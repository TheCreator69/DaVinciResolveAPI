# DaVinci Resolve - Unofficial Scripting Documentation

This document is a formatted copy of the official BlackmagicDesign DaVinci Resolve scripting documentation.

**WARNING: Keep in mind that this document might contain errors and might not be up to date with the current Resolve version. If in doubt, always consult the official Resolve documentation provided by BlackmagicDesign.**

*Last Updated: 3 October 2024*

In this package, you will find a brief introduction to the Scripting API for DaVinci Resolve Studio. Apart from this README.txt file, this package contains folders containing the basic import
modules for scripting access (`DaVinciResolve.py`) and some representative examples.

From v16.2.0 onwards, the `nodeIndex` parameters accepted by `SetLUT()` and `SetCDL()` are 1-based instead of 0-based, i.e. 1 <= `nodeIndex` <= total number of nodes.

Overview
--------
As with Blackmagic Fusion scripts, user scripts written in Lua and Python programming languages are supported. By default, scripts can be invoked from the Console window in the Fusion page,
or via command line. This permission can be changed in Resolve Preferences, to be only from Console, or to be invoked from the local network. Please be aware of the security implications when
allowing scripting access from outside of the Resolve application.

Prerequisites
-------------
DaVinci Resolve scripting requires one of the following to be installed (for all users):

    Lua 5.1
    Python >= 3.6 64-bit
    Python 2.7 64-bit

Using a script
--------------
DaVinci Resolve needs to be running for a script to be invoked.

For a Resolve script to be executed from an external folder, the script needs to know of the API location.
You may need to set the these environment variables to allow for your Python installation to pick up the appropriate dependencies as shown below:

    Mac OS X:
    RESOLVE_SCRIPT_API="/Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting"
    RESOLVE_SCRIPT_LIB="/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Libraries/Fusion/fusionscript.so"
    PYTHONPATH="$PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/"

    Windows:
    RESOLVE_SCRIPT_API="%PROGRAMDATA%\Blackmagic Design\DaVinci Resolve\Support\Developer\Scripting"
    RESOLVE_SCRIPT_LIB="C:\Program Files\Blackmagic Design\DaVinci Resolve\fusionscript.dll"
    PYTHONPATH="%PYTHONPATH%;%RESOLVE_SCRIPT_API%\Modules\"

    Linux:
    RESOLVE_SCRIPT_API="/opt/resolve/Developer/Scripting"
    RESOLVE_SCRIPT_LIB="/opt/resolve/libs/Fusion/fusionscript.so"
    PYTHONPATH="$PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/"
    (Note: For standard ISO Linux installations, the path above may need to be modified to refer to /home/resolve instead of /opt/resolve)

As with Fusion scripts, Resolve scripts can also be invoked via the menu and the Console.

On startup, DaVinci Resolve scans the subfolders in the directories shown below and enumerates the scripts found in the Workspace application menu under Scripts.  
Place your script under Utility to be listed in all pages, under Comp or Tool to be available in the Fusion page or under folders for individual pages (Edit, Color or Deliver).  
Scripts under Deliver are additionally listed under render jobs.  
Placing your script here and invoking it from the menu is the easiest way to use scripts.

    Mac OS X:
      - All users: /Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Scripts
      - Specific user:  /Users/<UserName>/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Scripts
    Windows:
      - All users: %PROGRAMDATA%\Blackmagic Design\DaVinci Resolve\Fusion\Scripts
      - Specific user: %APPDATA%\Roaming\Blackmagic Design\DaVinci Resolve\Support\Fusion\Scripts
    Linux:
      - All users: /opt/resolve/Fusion/Scripts  (or /home/resolve/Fusion/Scripts/ depending on installation)
      - Specific user: $HOME/.local/share/DaVinciResolve/Fusion/Scripts

The interactive Console window allows for an easy way to execute simple scripting commands, to query or modify properties, and to test scripts. The console accepts commands in Python 2.7, Python 3.6
and Lua and evaluates and executes them immediately. For more information on how to use the Console, please refer to the DaVinci Resolve User Manual.

This example Python script creates a simple project:

    #!/usr/bin/env python
    import DaVinciResolveScript as dvr_script
    resolve = dvr_script.scriptapp("Resolve")
    fusion = resolve.Fusion()
    projectManager = resolve.GetProjectManager()
    projectManager.CreateProject("Hello World")

The resolve object is the fundamental starting point for scripting via Resolve. As a native object, it can be inspected for further scriptable properties - using table iteration and "getmetatable"
in Lua and dir, help etc in Python (among other methods). A notable scriptable object above is fusion - it allows access to all existing Fusion scripting functionality.


Running DaVinci Resolve in headless mode
----------------------------------------
DaVinci Resolve can be launched in a headless mode without the user interface using the `-nogui` command line option. When DaVinci Resolve is launched using this option, the user interface is disabled.
However, the various scripting APIs will continue to work as expected.

DaVinci Resolve API
-------------------
Some commonly used API functions are described below. As with the `Resolve` object, each object is inspectable for properties and functions.

### `Resolve` Methods

| Method                                            | Return Type         | Comment                                                                                                                                                                                                                                                                                                      |
| ------------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Fusion()`                                        | `Fusion`            | Returns the `Fusion` object. Starting point for Fusion scripts.                                                                                                                                                                                                                                                |
| `GetMediaStorage()`                               | `MediaStorage`      | Returns the `MediaStorage` object to query and act on media locations.                                                                                                                                                                                                                                        |
| `GetProjectManager()`                             | `ProjectManager`    | Returns the `ProjectManager` object for currently open database.                                                                                                                                                                                                                                              |
| `OpenPage(pageName)`                              | `Bool`              | Switches to indicated page in DaVinci Resolve. Input can be one of (`"media"`, `"cut"`, `"edit"`, `"fusion"`, `"color"`, `"fairlight"`, `"deliver"`).                                                                                                                                                               |
| `GetCurrentPage()`                                | `String`            | Returns the page currently displayed in the main window. Returned value can be one of (`"media"`, `"cut"`, `"edit"`, `"fusion"`, `"color"`, `"fairlight"`, `"deliver"`, `None`).                                                                                   |
| `GetProductName()`                                | `string`            | Returns product name.                                                                                                                                                                                                                                                                                         |
| `GetVersion()`                                    | `[version field]`   | Returns list of product version fields in `[major, minor, patch, build, suffix]` format.                                                                                                                                                                                                                      |
| `GetVersionString()`                              | `string`            | Returns product version in `"major.minor.patch[suffix].build"` format.                                                                                                                                                                                                                                         |
| `LoadLayoutPreset(presetName)`                    | `Bool`              | Loads UI layout from saved preset named `presetName`.                                                                                                                                                                                                                                                  |
| `UpdateLayoutPreset(presetName)`                  | `Bool`              | Overwrites preset named `presetName` with current UI layout.                                                                                                                                                                                                                                           |
| `ExportLayoutPreset(presetName, presetFilePath)`  | `Bool`              | Exports preset named `presetName` to path `presetFilePath`.                                                                                                                                                                                                                                            |
| `DeleteLayoutPreset(presetName)`                  | `Bool`              | Deletes preset named `presetName`.                                                                                                                                                                                                                                                                             |
| `SaveLayoutPreset(presetName)`                    | `Bool`              | Saves current UI layout as a preset named `presetName`.                                                                                                                                                                                                                                                     |
| `ImportLayoutPreset(presetFilePath, presetName)`  | `Bool`              | Imports preset from path `presetFilePath`. The optional argument `presetName` specifies how the preset shall be named. If not specified, the preset is named based on the filename.                                                                                                                  |
| `Quit()`                                          | `None`              | Quits the Resolve App.                                                                                                                                                                                                                                                                                           |
| `ImportRenderPreset(presetPath)`                  | `Bool`              | Import a preset from `presetPath` (`string`) and set it as current preset for rendering.                                                                                                                                                                                                                     |
| `ExportRenderPreset(presetName, exportPath)`      | `Bool`              | Export a preset to a given path (`string`) if `presetName` (`string`) exists.                                                                                                                                                                                                                                    |
| `ImportBurnInPreset(presetPath)`                  | `Bool`              | Import a data burn in preset from a given `presetPath` (`string`).                                                                                                                                                                                                                                              |
| `ExportBurnInPreset(presetName, exportPath)`      | `Bool`              | Export a data burn in preset to a given path (`string`) if `presetName` (`string`) exists.                                                                                                                                                                                                                       |
| `GetKeyframeMode()`                               | `keyframeMode`      | Returns the currently set keyframe mode (`int`). Refer to section 'Keyframe Mode information' below for details.                                                                                                                                                                                               |
| `SetKeyframeMode(keyframeMode)`                   | `Bool`              | Returns `True` when `keyframeMode` (`enum`) is successfully set. Refer to section 'Keyframe Mode information' below for details.                                                                                                                                                                                |


### `ProjectManager` Methods

| Method                                                                 | Return Type          | Comment                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------------------------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ArchiveProject(projectName, filePath, isArchiveSrcMedia=True, isArchiveRenderCache=True, isArchiveProxyMedia=False)` | `Bool`               | Archives project to provided file path with the configuration as provided by the optional arguments.                                                                                                                                                                                                                                 |
| `CreateProject(projectName)`                                           | `Project`            | Creates and returns a project if `projectName` (`string`) is unique, and `None` if it is not.                                                                                                                                                                                                                                       |
| `DeleteProject(projectName)`                                           | `Bool`               | Delete project in the current folder if not currently loaded.                                                                                                                                                                                                                                                                      |
| `LoadProject(projectName)`                                             | `Project`            | Loads and returns the project with name = `projectName` (`string`) if there is a match found, and `None` if there is no matching Project.                                                                                                                                                                                             |
| `GetCurrentProject()`                                                  | `Project`            | Returns the currently loaded Resolve project.                                                                                                                                                                                                                                                                                      |
| `SaveProject()`                                                        | `Bool`               | Saves the currently loaded project with its own name. Returns `True` if successful.                                                                                                                                                                                                                                                  |
| `CloseProject(project)`                                                | `Bool`               | Closes the specified project without saving.                                                                                                                                                                                                                                                                                        |
| `CreateFolder(folderName)`                                             | `Bool`               | Creates a folder if `folderName` (`string`) is unique.                                                                                                                                                                                                                                                                                |
| `DeleteFolder(folderName)`                                             | `Bool`               | Deletes the specified folder if it exists. Returns `True` in case of success.                                                                                                                                                                                                                                                         |
| `GetProjectListInCurrentFolder()`                                      | `[project names...]` | Returns a list of project names in current folder.                                                                                                                                                                                                                                                                                |
| `GetFolderListInCurrentFolder()`                                       | `[folder names...]`  | Returns a list of folder names in current folder.                                                                                                                                                                                                                                                                                |
| `GotoRootFolder()`                                                     | `Bool`               | Opens root folder in database.                                                                                                                                                                                                                                                                                                      |
| `GotoParentFolder()`                                                   | `Bool`               | Opens parent folder of current folder in database if current folder has parent.                                                                                                                                                                                                                                                      |
| `GetCurrentFolder()`                                                   | `string`             | Returns the current folder name.                                                                                                                                                                                                                                                                                                  |
| `OpenFolder(folderName)`                                               | `Bool`               | Opens folder under given name.                                                                                                                                                                                                                                                                                                      |
| `ImportProject(filePath, projectName=None)`                            | `Bool`               | Imports a project from the file path provided with given project name, if any. Returns `True` if successful.                                                                                                                                                                                                                    |
| `ExportProject(projectName, filePath, withStillsAndLUTs=True)`         | `Bool`               | Exports project to provided file path, including stills and LUTs if `withStillsAndLUTs` is `True` (enabled by default). Returns `True` in case of success.                                                                                                                                                                           |
| `RestoreProject(filePath, projectName=None)`                           | `Bool`               | Restores a project from the file path provided with given project name, if any. Returns `True` if successful.                                                                                                                                                                                                                     |
| `GetCurrentDatabase()`                                                 | `{dbInfo}`           | Returns a dictionary (with keys `'DbType'`, `'DbName'` and optional `'IpAddress'`) corresponding to the current database connection.                                                                                                                                                                                               |
| `GetDatabaseList()`                                                    | `[{dbInfo}]`         | Returns a list of dictionary items (with keys `'DbType'`, `'DbName'` and optional `'IpAddress'`) corresponding to all the databases added to Resolve.                                                                                                                                                                                |
| `SetCurrentDatabase({dbInfo})`                                         | `Bool`               | Switches current database connection to the database specified by the keys below, and closes any open project. `'DbType'`: `'Disk'` or `'PostgreSQL'` (`string`). `'DbName'`: database name (`string`). `'IpAddress'`: IP address of the PostgreSQL server (`string`, optional key - defaults to `'127.0.0.1'`). |
| `CreateCloudProject({cloudSettings})`                                  | `Project`            | Creates and returns a cloud project. `{cloudSettings}`: Check `'Cloud Projects Settings'` subsection below for more information.                                                                                                                                         |
| `ImportCloudProject(filePath, {cloudSettings})`                        | `Bool`               | Returns `True` if import cloud project is successful; `False` otherwise. `'filePath'`: `String`; file path of file to import. `{cloudSettings}`: Check `'Cloud Projects Settings'` subsection below for more information.                                                                                                                                                   |
| `RestoreCloudProject(folderPath, {cloudSettings})`                     | `Bool`               | Returns `True` if restore cloud project is successful; `False` otherwise. `'folderPath'`: `String`; path of folder to restore. `{cloudSettings}`: Check `'Cloud Projects Settings'` subsection below for more information.                                                                                                                                                   |
| ~~`GetProjectsInCurrentFolder()`~~                                      | ~~`{project names...}`~~ | ~~Returns a dictionary of project names in the current folder.~~                                                                                                                                                                                                                                                                      |
| ~~`GetFoldersInCurrentFolder()`~~                                       | ~~`{folder names...}`~~  | ~~Returns a dictionary of folder names in the current folder.~~                                                                                                                                                                                                                                                                      |

### `Project` Methods

| Method                                                                 | Return Type          | Comment                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------------------------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GetMediaPool()`                                                       | `MediaPool`          | Returns the Media Pool object.                                                                                                                                                                                                                                                                                                     |
| `GetTimelineCount()`                                                   | `int`                | Returns the number of timelines currently present in the project.                                                                                                                                                                                                                                                               |
| `GetTimelineByIndex(idx)`                                              | `Timeline`           | Returns timeline at the given index, 1 <= `idx` <= `project.GetTimelineCount()`.                                                                                                                                                                                                                                                    |
| `GetCurrentTimeline()`                                                 | `Timeline`           | Returns the currently loaded timeline.                                                                                                                                                                                                                                                                                           |
| `SetCurrentTimeline(timeline)`                                         | `Bool`               | Sets given timeline as current timeline for the project. Returns `True` if successful.                                                                                                                                                                                                                                               |
| `GetGallery()`                                                         | `Gallery`            | Returns the Gallery object.                                                                                                                                                                                                                                                                                                          |
| `GetName()`                                                            | `string`             | Returns project name.                                                                                                                                                                                                                                                                                                              |
| `SetName(projectName)`                                                 | `Bool`               | Sets project name if given `projectName` (`string`) is unique.                                                                                                                                                                                                                                                                    |
| `GetPresetList()`                                                      | `[presets...]`       | Returns a list of presets and their information.                                                                                                                                                                                                                                                                                  |
| `SetPreset(presetName)`                                                | `Bool`               | Sets preset by given `presetName` (`string`) into project.                                                                                                                                                                                                                                                                        |
| `AddRenderJob()`                                                       | `string`             | Adds a render job based on current render settings to the render queue. Returns a unique job id (`string`) for the new render job.                                                                                                                                                                                                  |
| `DeleteRenderJob(jobId)`                                               | `Bool`               | Deletes render job for input job id (`string`).                                                                                                                                                                                                                                                                                   |
| `DeleteAllRenderJobs()`                                                | `Bool`               | Deletes all render jobs in the queue.                                                                                                                                                                                                                                                                                           |
| `GetRenderJobList()`                                                   | `[render jobs...]`   | Returns a list of render jobs and their information.                                                                                                                                                                                                                                                                                |
| `GetRenderPresetList()`                                                | `[presets...]`       | Returns a list of render presets and their information.                                                                                                                                                                                                                                                                            |
| `StartRendering(jobId1, jobId2, ...)`                                  | `Bool`               | Starts rendering jobs indicated by the input job ids.                                                                                                                                                                                                                                                                            |
| `StartRendering([jobIds...], isInteractiveMode=False)`                 | `Bool`               | Starts rendering jobs indicated by the input job ids. The optional `isInteractiveMode`, when set, enables error feedback in the UI during rendering.                                                                                                                                                                              |
| `StartRendering(isInteractiveMode=False)`                              | `Bool`               | Starts rendering all queued render jobs. The optional `isInteractiveMode`, when set, enables error feedback in the UI during rendering.                                                                                                                                                                                          |
| `StopRendering()`                                                      | `None`               | Stops any current render processes.                                                                                                                                                                                                                                                                                               |
| `IsRenderingInProgress()`                                              | `Bool`               | Returns `True` if rendering is in progress.                                                                                                                                                                                                                                                                                       |
| `LoadRenderPreset(presetName)`                                         | `Bool`               | Sets a preset as current preset for rendering if `presetName` (`string`) exists.                                                                                                                                                                                                                                                   |
| `SaveAsNewRenderPreset(presetName)`                                    | `Bool`               | Creates new render preset by given name if `presetName` (`string`) is unique.                                                                                                                                                                                                                                                       |
| `SetRenderSettings({settings})`                                        | `Bool`               | Sets given settings for rendering. `Settings` is a dict, with support for the keys: Refer to "Looking up render settings" section for information on supported settings.                                                                                                                                                         |
| `GetRenderJobStatus(jobId)`                                            | `{status info}`      | Returns a dict with job status and completion percentage of the job by given `jobId` (`string`).                                                                                                                                                                           |
| `GetSetting(settingName)`                                              | `string`             | Returns value of project setting (indicated by `settingName`, `string`). Check the section below for more information.                                                                                                                                                                                                               |
| `SetSetting(settingName, settingValue)`                                | `Bool`               | Sets the project setting (indicated by `settingName`, `string`) to the value (`settingValue`, `string`). Check the section below for more information.                                                                                                                                                                              |
| `GetRenderFormats()`                                                   | `{render formats...}` | Returns a dict (`format` -> `file extension`) of available render formats.                                                                                                                                                                                                                                                          |
| `GetRenderCodecs(renderFormat)`                                        | `{render codecs...}` | Returns a dict (`codec description` -> `codec name`) of available codecs for given render format (`string`).                                                                                                                                                                                                                       |
| `GetCurrentRenderFormatAndCodec()`                                     | `{format, codec}`    | Returns a dict with currently selected format `format` and render codec `codec`.                                                                                                                                                                                                                                                   |
| `SetCurrentRenderFormatAndCodec(format, codec)`                        | `Bool`               | Sets given render format (`string`) and render codec (`string`) as options for rendering.                                                                                                                                                                                                                                           |
| `GetCurrentRenderMode()`                                               | `int`                | Returns the render mode: 0 - Individual clips, 1 - Single clip.                                                                                                                                                                                                                                                                    |
| `SetCurrentRenderMode(renderMode)`                                     | `Bool`               | Sets the render mode. Specify `renderMode = 0` for Individual clips, 1 for Single clip.                                                                                                                                                                                                                                             |
| `GetRenderResolutions(format, codec)`                                  | `[{Resolution}]`     | Returns list of resolutions applicable for the given render format (`string`) and render codec (`string`). Returns full list of resolutions if no argument is provided. Each element in the list is a dictionary with 2 keys "Width" and "Height".                        |
| `RefreshLUTList()`                                                     | `Bool`               | Refreshes LUT List.                                                                                                                                                                                                                                                                                                                 |
| `GetUniqueId()`                                                        | `string`             | Returns a unique ID for the project item.                                                                                                                                                                                                                                                                                        |
| `InsertAudioToCurrentTrackAtPlayhead(mediaPath, startOffsetInSamples, durationInSamples)` | `Bool`               | Inserts the media specified by `mediaPath` (`string`) with `startOffsetInSamples` (`int`) and `durationInSamples` (`int`) at the playhead on a selected track on the Fairlight page. Returns `True` if successful, otherwise `False`.                                          |
| `LoadBurnInPreset(presetName)`                                         | `Bool`               | Loads user-defined data burn-in preset for project when supplied `presetName` (`string`). Returns `True` if successful.                                                                                                                                                                                                         |
| `ExportCurrentFrameAsStill(filePath)`                                  | `Bool`               | Exports current frame as still to supplied `filePath`. `filePath` must end in valid export file format. Returns `True` if successful, `False` otherwise.                                                                                                                                                                            |
| `GetColorGroupsList()`                                                 | `[ColorGroups...]`   | Returns a list of all group objects in the timeline.                                                                                                                                                                                                                                                                               |
| `AddColorGroup(groupName)`                                             | `ColorGroup`         | Creates a new `ColorGroup`. `groupName` must be a unique `string`.                                                                                                                                                                                                                                                                |
| `DeleteColorGroup(colorGroup)`                                         | `Bool`               | Deletes the given color group and sets clips to ungrouped.                                                                                                                                                                                                                                                                         |
| ~~`GetPresets()`~~                                                     | ~~`{presets...}`~~       | ~~Returns a dictionary of presets and their information.~~                                                                                                                                                                                                                                                                            |
| ~~`GetRenderJobs()`~~                                                  | ~~`{render jobs...}`~~   | ~~Returns a dictionary of render jobs and their information.~~                                                                                                                                                                                                                                                                         |
| ~~`GetRenderPresets()`~~                                               | ~~`{presets...}`~~       | ~~Returns a dictionary of render presets and their information.~~                                                                                                                                                                                                                                                                    |

### `MediaStorage` Methods

| Method                                                            | Return Type          | Comment                                                                                                                                                                                                                                                |
| ----------------------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `GetMountedVolumeList()`                                           | `[paths...]`         | Returns list of folder paths corresponding to mounted volumes displayed in Resolve’s Media Storage.                                                                                                                                                        |
| `GetSubFolderList(folderPath)`                                     | `[paths...]`         | Returns list of folder paths in the given absolute folder path.                                                                                                                                                                                           |
| `GetFileList(folderPath)`                                          | `[paths...]`         | Returns list of media and file listings in the given absolute folder path. Note that media listings may be logically consolidated entries.                                                                                                              |
| `RevealInStorage(path)`                                            | `Bool`               | Expands and displays given file/folder path in Resolve’s Media Storage.                                                                                                                                                                                   |
| `AddItemListToMediaPool(item1, item2, ...)`                        | `[clips...]`         | Adds specified file/folder paths from Media Storage into current Media Pool folder. Input is one or more file/folder paths. Returns a list of the `MediaPoolItems` created.                                                                             |
| `AddItemListToMediaPool([items...])`                               | `[clips...]`         | Adds specified file/folder paths from Media Storage into current Media Pool folder. Input is an array of file/folder paths. Returns a list of the `MediaPoolItems` created.                                                                          |
| `AddItemListToMediaPool([{itemInfo}, ...])`                        | `[clips...]`         | Adds list of `itemInfos` specified as dict of "media", "startFrame" (int), "endFrame" (int) from Media Storage into current Media Pool folder. Returns a list of the `MediaPoolItems` created.                                                        |
| `AddClipMattesToMediaPool(MediaPoolItem, [paths], stereoEye)`      | `Bool`               | Adds specified media files as mattes for the specified `MediaPoolItem`. `StereoEye` is an optional argument for specifying which eye to add the matte to for stereo clips ("left" or "right"). Returns `True` if successful. |
| `AddTimelineMattesToMediaPool([paths])`                            | `[MediaPoolItems]`   | Adds specified media files as timeline mattes in current media pool folder. Returns a list of created `MediaPoolItems`.                                                                                                                                  |
| ~~`GetMountedVolumes()`~~                                           | ~~`{paths...}`~~      | ~~Returns a dictionary of folder paths corresponding to mounted volumes in Resolve's Media Storage.~~                                                                                                                                                       |
| ~~`GetSubFolders(folderPath)`~~                                     | ~~`{paths...}`~~      | ~~Returns a dictionary of folder paths within the specified absolute folder path.~~                                                                                                                                                                       |
| ~~`GetFiles(folderPath)`~~                                          | ~~`{paths...}`~~      | ~~Returns a dictionary of media and file listings in the specified absolute folder path. Media listings may be logically consolidated entries.~~                                                                                                         |
| ~~`AddItemsToMediaPool(item1, item2, ...)`~~                        | ~~`{clips...}`~~      | ~~Adds specified file/folder paths from Media Storage into the current Media Pool folder. Returns a dictionary of created `MediaPoolItems`.~~                                                                                                           |
| ~~`AddItemsToMediaPool([items...])`~~                               | ~~`{clips...}`~~      | ~~Adds specified file/folder paths from Media Storage into the current Media Pool folder as an array. Returns a dictionary of created `MediaPoolItems`.~~                                                                                               |


### `MediaPool` Methods

| Method                                                            | Return Type          | Comment                                                                                                                                                                                                                                                                 |
| ----------------------------------------------------------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GetRootFolder()`                                                 | `Folder`             | Returns root Folder of Media Pool                                                                                                                                                                                                                                      |
| `AddSubFolder(folder, name)`                                      | `Folder`             | Adds new subfolder under specified Folder object with the given name.                                                                                                                                                                                                  |
| `RefreshFolders()`                                                | `Bool`               | Updates the folders in collaboration mode                                                                                                                                                                                                                               |
| `CreateEmptyTimeline(name)`                                       | `Timeline`           | Adds new timeline with given name.                                                                                                                                                                                                                                      |
| `AppendToTimeline(clip1, clip2, ...)`                             | `[TimelineItem]`     | Appends specified `MediaPoolItem` objects in the current timeline. Returns the list of appended timelineItems.                                                                                                                                                          |
| `AppendToTimeline([clips])`                                       | `[TimelineItem]`     | Appends specified `MediaPoolItem` objects in the current timeline. Returns the list of appended timelineItems.                                                                                                                                                          |
| `AppendToTimeline([{clipInfo}, ...])`                             | `[TimelineItem]`     | Appends list of `clipInfos` specified as dict of "mediaPoolItem", "startFrame" (float/int), "endFrame" (float/int), (optional) "mediaType" (int; 1 - Video only, 2 - Audio only), "trackIndex" (int) and "recordFrame" (float/int). Returns the list of appended timelineItems. |
| `CreateTimelineFromClips(name, clip1, clip2,...)`                 | `Timeline`           | Creates new timeline with specified name, and appends the specified `MediaPoolItem` objects.                                                                                                                                                                            |
| `CreateTimelineFromClips(name, [clips])`                          | `Timeline`           | Creates new timeline with specified name, and appends the specified `MediaPoolItem` objects.                                                                                                                                                                            |
| `CreateTimelineFromClips(name, [{clipInfo}])`                     | `Timeline`           | Creates new timeline with specified name, appending the list of `clipInfos` specified as a dict of "mediaPoolItem", "startFrame" (float/int), "endFrame" (float/int), "recordFrame" (float/int).                                                                      |
| `ImportTimelineFromFile(filePath, {importOptions})`               | `Timeline`           | Creates timeline based on parameters within given file (AAF/EDL/XML/FCPXML/DRT/ADL/OTIO) and optional importOptions dict, with support for the keys: <br>`timelineName`: string, specifies the name of the timeline to be created. Not valid for DRT import. <br>`importSourceClips`: Bool, specifies whether source clips should be imported, True by default. Not valid for DRT import. <br>`sourceClipsPath`: string, specifies a filesystem path to search for source clips if the media is inaccessible in their original path and if `importSourceClips` is True. <br>`sourceClipsFolders`: List of Media Pool folder objects to search for source clips if the media is not present in current folder and if `importSourceClips` is False. Not valid for DRT import. <br>`interlaceProcessing`: Bool, specifies whether to enable interlace processing on the imported timeline being created. valid only for AAF import. |
| `DeleteTimelines([timeline])`                                     | `Bool`               | Deletes specified timelines in the media pool.                                                                                                                                                                                                                         |
| `GetCurrentFolder()`                                              | `Folder`             | Returns currently selected Folder.                                                                                                                                                                                                                                      |
| `SetCurrentFolder(Folder)`                                        | `Bool`               | Sets current folder by given Folder.                                                                                                                                                                                                                                   |
| `DeleteClips([clips])`                                            | `Bool`               | Deletes specified clips or timeline mattes in the media pool.                                                                                                                                                                                                         |
| `ImportFolderFromFile(filePath, sourceClipsPath="")`              | `Bool`               | Returns true if import from given DRB filePath is successful, false otherwise. `sourceClipsPath` is a string that specifies a filesystem path to search for source clips if the media is inaccessible in their original path, empty by default.                         |
| `DeleteFolders([subfolders])`                                     | `Bool`               | Deletes specified subfolders in the media pool.                                                                                                                                                                                                                       |
| `MoveClips([clips], targetFolder)`                                | `Bool`               | Moves specified clips to target folder.                                                                                                                                                                                                                               |
| `MoveFolders([folders], targetFolder)`                            | `Bool`               | Moves specified folders to target folder.                                                                                                                                                                                                                             |
| `GetClipMatteList(MediaPoolItem)`                                 | `[paths]`            | Get mattes for specified `MediaPoolItem`, as a list of paths to the matte files.                                                                                                                                                                                        |
| `GetTimelineMatteList(Folder)`                                    | `[MediaPoolItems]`   | Get mattes in specified Folder, as list of `MediaPoolItems`.                                                                                                                                                                                                         |
| `DeleteClipMattes(MediaPoolItem, [paths])`                        | `Bool`               | Delete mattes based on their file paths, for specified `MediaPoolItem`. Returns True on success.                                                                                                                                                                        |
| `RelinkClips([MediaPoolItem], folderPath)`                        | `Bool`               | Update the folder location of specified media pool clips with the specified folder path.                                                                                                                                                                               |
| `UnlinkClips([MediaPoolItem])`                                    | `Bool`               | Unlink specified media pool clips.                                                                                                                                                                                                                                     |
| `ImportMedia([items...])`                                         | `[MediaPoolItems]`   | Imports specified file/folder paths into current Media Pool folder. Input is an array of file/folder paths. Returns a list of the `MediaPoolItems` created.                                                                                                       |
| `ImportMedia([{clipInfo}])`                                       | `[MediaPoolItems]`   | Imports file path(s) into current Media Pool folder as specified in list of `clipInfo` dict. Returns a list of the `MediaPoolItems` created. Each `clipInfo` gets imported as one `MediaPoolItem` unless 'Show Individual Frames' is turned on.                        |
| `ExportMetadata(fileName, [clips])`                               | `Bool`               | Exports metadata of specified clips to 'fileName' in CSV format. If no clips are specified, all clips from media pool will be used.                                                                                                                                 |
| `GetUniqueId()`                                                   | `string`             | Returns a unique ID for the media pool.                                                                                                                                                                                                                                |
| `CreateStereoClip(LeftMediaPoolItem, RightMediaPoolItem)`         | `MediaPoolItem`      | Takes in two existing media pool items and creates a new 3D stereoscopic media pool entry replacing the input media in the media pool.                                                                                                                                  |
| `GetSelectedClips()`                                              | `[MediaPoolItems]`   | Returns the current selected `MediaPoolItems`.                                                                                                                                                                                                                        |
| `SetSelectedClip(MediaPoolItem)`                                  | `Bool`               | Sets the selected `MediaPoolItem` to the given `MediaPoolItem`.                                                                                                                                                                                                     |

### `Folder` Methods

| Method                                               | Return Type         | Comment                                                                                                                                                                      |
| ---------------------------------------------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GetClipList()`                                       | `[clips...]`         | Returns a list of clips (items) within the folder.                                                                                                                                                                                 |
| `GetName()`                                           | `string`             | Returns the media folder name.                                                                                                                                                                                                       |
| `GetSubFolderList()`                                  | `[folders...]`       | Returns a list of subfolders in the folder.                                                                                                                                                                                           |
| `GetIsFolderStale()`                                  | `bool`               | Returns true if the folder is stale in collaboration mode, false otherwise.                                                                                                                                                           |
| `GetUniqueId()`                                       | `string`             | Returns a unique ID for the media pool folder.                                                                                                                                                                                        |
| `Export(filePath)`                                    | `bool`               | Returns true if export of DRB folder to filePath is successful, false otherwise.                                                                                                                                                      |
| `TranscribeAudio()`                                   | `Bool`               | Transcribes audio of the `MediaPoolItems` within the folder and nested folders. Returns True if successful; False otherwise.                                                                                                       |
| `ClearTranscription()`                                | `Bool`               | Clears audio transcription of the `MediaPoolItems` within the folder and nested folders. Returns True if successful; False otherwise.                                                                                             |
| ~~`GetClips()`~~                                       | ~~`{clips...}`~~      | ~~Returns a dictionary of clips within the folder.~~                                                                                                                        |
| ~~`GetSubFolders()`~~                                  | ~~`{folders...}`~~    | ~~Returns a dictionary of subfolders within the folder.~~                                                                                                                   |

### `MediaPoolItem` Methods

| Method                                               | Return Type               | Comment                                                                                                                                                                      |
| ---------------------------------------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GetName()`                                           | `string`                  | Returns the clip name.                                                                                                                                                        |
| `GetMetadata(metadataType=None)`                      | `string/dict`             | Returns the metadata value for the key 'metadataType'. If no argument is specified, a dict of all set metadata properties is returned.                                       |
| `SetMetadata(metadataType, metadataValue)`            | `Bool`                     | Sets the given metadata to `metadataValue` (string). Returns True if successful.                                                                                              |
| `SetMetadata({metadata})`                             | `Bool`                     | Sets the item metadata with the specified 'metadata' dict. Returns True if successful.                                                                                       |
| `GetThirdPartyMetadata(metadataType=None)`            | `string/dict`             | Returns the third-party metadata value for the key 'metadataType'. If no argument is specified, a dict of all set third-party metadata properties is returned.              |
| `SetThirdPartyMetadata(metadataType, metadataValue)`  | `Bool`                     | Sets/Add the given third-party metadata to `metadataValue` (string). Returns True if successful.                                                                             |
| `SetThirdPartyMetadata({metadata})`                   | `Bool`                     | Sets/Add the item third-party metadata with the specified 'metadata' dict. Returns True if successful.                                                                     |
| `GetMediaId()`                                        | `string`                  | Returns the unique ID for the `MediaPoolItem`.                                                                                                                                 |
| `AddMarker(frameId, color, name, note, duration, customData)` | `Bool`                     | Creates a new marker at the given `frameId` position and with the given marker information. 'customData' is optional and helps to attach user-specific data to the marker.    |
| `GetMarkers()`                                        | `{markers...}`            | Returns a dict (frameId -> {information}) of all markers and dicts with their information. Example of output format: {96.0: {'color': 'Green', 'duration': 1.0, 'note': '', 'name': 'Marker 1', 'customData': ''}, ...}. |
| `GetMarkerByCustomData(customData)`                   | `{markers...}`            | Returns marker {information} for the first matching marker with the specified `customData`.                                                                                 |
| `UpdateMarkerCustomData(frameId, customData)`         | `Bool`                     | Updates `customData` (string) for the marker at the given `frameId` position. CustomData is not exposed via UI and is useful for scripting developers to attach user-specific data to markers. |
| `GetMarkerCustomData(frameId)`                        | `string`                  | Returns `customData` string for the marker at the given `frameId` position.                                                                                                  |
| `DeleteMarkersByColor(color)`                         | `Bool`                     | Deletes all markers of the specified color from the `MediaPoolItem`. "All" as argument deletes all color markers.                                                            |
| `DeleteMarkerAtFrame(frameNum)`                       | `Bool`                     | Deletes marker at the given frame number from the `MediaPoolItem`.                                                                                                          |
| `DeleteMarkerByCustomData(customData)`                | `Bool`                     | Deletes the first matching marker with the specified `customData`.                                                                                                          |
| `AddFlag(color)`                                      | `Bool`                     | Adds a flag with the given color (string).                                                                                                                                     |
| `GetFlagList()`                                       | `[colors...]`             | Returns a list of flag colors assigned to the item.                                                                                                                           |
| `ClearFlags(color)`                                   | `Bool`                     | Clears the flag of the given color if one exists. An "All" argument is supported and clears all flags.                                                                       |
| `GetClipColor()`                                      | `string`                  | Returns the item color as a string.                                                                                                                                           |
| `SetClipColor(colorName)`                             | `Bool`                     | Sets the item color based on the `colorName` (string).                                                                                                                       |
| `ClearClipColor()`                                    | `Bool`                     | Clears the item color.                                                                                                                                                        |
| `GetClipProperty(propertyName=None)`                  | `string/dict`             | Returns the property value for the key 'propertyName'. If no argument is specified, a dict of all clip properties is returned. Check the section below for more information. |
| `SetClipProperty(propertyName, propertyValue)`        | `Bool`                     | Sets the given property to `propertyValue` (string). Check the section below for more information.                                                                          |
| `LinkProxyMedia(proxyMediaFilePath)`                  | `Bool`                     | Links proxy media located at path specified by the argument `proxyMediaFilePath` with the current clip. `proxyMediaFilePath` should be an absolute clip path.              |
| `UnlinkProxyMedia()`                                  | `Bool`                     | Unlinks any proxy media associated with the clip.                                                                                                                             |
| `ReplaceClip(filePath)`                               | `Bool`                     | Replaces the underlying asset and metadata of the `MediaPoolItem` with the specified absolute clip path.                                                                   |
| `GetUniqueId()`                                       | `string`                  | Returns a unique ID for the `MediaPoolItem`.                                                                                                                                   |
| `TranscribeAudio()`                                   | `Bool`                     | Transcribes audio of the `MediaPoolItem`. Returns True if successful; False otherwise.                                                                                       |
| `ClearTranscription()`                                | `Bool`                     | Clears audio transcription of the `MediaPoolItem`. Returns True if successful; False otherwise.                                                                            |
| `GetAudioMapping()`                                   | `json formatted string`   | Returns a string with `MediaPoolItem`'s audio mapping information. Check 'Audio Mapping' section below for more information.                                                 |
| ~~`GetFlags()`~~                                       | ~~`{colors...}`~~          | ~~Returns a dictionary of flag colors assigned to the item.~~                                                                                                               |

### `Timeline` Methods

| Method                                                     | Return Type           | Comment |
| ---------------------------------------------------------- | --------------------- | ------- |
| `GetName()`                                                | `string`              | Returns the timeline name. |
| `SetName(timelineName)`                                     | `Bool`                | Sets the timeline name if `timelineName` (string) is unique. Returns True if successful. |
| `GetStartFrame()`                                           | `int`                 | Returns the frame number at the start of the timeline. |
| `GetEndFrame()`                                             | `int`                 | Returns the frame number at the end of the timeline. |
| `SetStartTimecode(timecode)`                                | `Bool`                | Sets the start timecode of the timeline to the string 'timecode'. Returns True if successful. |
| `GetStartTimecode()`                                        | `string`              | Returns the start timecode for the timeline. |
| `GetTrackCount(trackType)`                                  | `int`                 | Returns the number of tracks for the given track type ("audio", "video", or "subtitle"). |
| `AddTrack(trackType, subTrackType)`                         | `Bool`                | Adds track of `trackType` ("video", "subtitle", "audio"). Optional argument `subTrackType` is used for "audio" track type. |
| `AddTrack(trackType, newTrackOptions)`                      | `Bool`                | Adds track of `trackType` ("video", "subtitle", "audio"). Optional `newTrackOptions` allows setting specific properties such as `audioType` and `index`. |
| `DeleteTrack(trackType, trackIndex)`                        | `Bool`                | Deletes track of `trackType` ("video", "subtitle", "audio") and given `trackIndex`. |
| `GetTrackSubType(trackType, trackIndex)`                    | `string`              | Returns an audio track's format. Returns a blank string for non-audio tracks. |
| `SetTrackEnable(trackType, trackIndex, Bool)`               | `Bool`                | Enables/Disables track with given `trackType` and `trackIndex`. |
| `GetIsTrackEnabled(trackType, trackIndex)`                  | `Bool`                | Returns True if track with given `trackType` and `trackIndex` is enabled, False otherwise. |
| `SetTrackLock(trackType, trackIndex, Bool)`                 | `Bool`                | Locks/Unlocks track with given `trackType` and `trackIndex`. |
| `GetIsTrackLocked(trackType, trackIndex)`                   | `Bool`                | Returns True if track with given `trackType` and `trackIndex` is locked, False otherwise. |
| `DeleteClips([timelineItems], Bool)`                        | `Bool`                | Deletes specified `TimelineItems` from the timeline, performing ripple delete if the second argument is True. |
| `SetClipsLinked([timelineItems], Bool)`                     | `Bool`                | Links or unlinks the specified `TimelineItems` depending on the second argument. |
| `GetItemListInTrack(trackType, index)`                      | `[items...]`          | Returns a list of timeline items on that track (based on `trackType` and `index`). |
| `AddMarker(frameId, color, name, note, duration, customData)`| `Bool`                | Creates a new marker at the given `frameId` position with the specified information. |
| `GetMarkers()`                                              | `{markers...}`        | Returns a dict of all markers with frameId as the key and marker information as the value. |
| `GetMarkerByCustomData(customData)`                         | `{markers...}`        | Returns marker information for the first matching marker with the specified `customData`. |
| `UpdateMarkerCustomData(frameId, customData)`               | `Bool`                | Updates the `customData` for the marker at the given `frameId` position. |
| `GetMarkerCustomData(frameId)`                              | `string`              | Returns `customData` for the marker at the given `frameId` position. |
| `DeleteMarkersByColor(color)`                               | `Bool`                | Deletes all timeline markers of the specified color. An "All" argument deletes all markers. |
| `DeleteMarkerAtFrame(frameNum)`                             | `Bool`                | Deletes the timeline marker at the given frame number. |
| `DeleteMarkerByCustomData(customData)`                      | `Bool`                | Deletes the first matching marker with the specified `customData`. |
| `ApplyGradeFromDRX(path, gradeMode, item1, item2, ...)`      | `Bool`                | Loads a still from the file path and applies a grade to Timeline Items with the specified `gradeMode`. |
| `ApplyGradeFromDRX(path, gradeMode, [items])`                | `Bool`                | Loads a still from the file path and applies a grade to the specified Timeline Items with the `gradeMode`. |
| `GetCurrentTimecode()`                                       | `string`              | Returns a string timecode for the current playhead position. |
| `SetCurrentTimecode(timecode)`                               | `Bool`                | Sets the current playhead position based on the input timecode. |
| `GetCurrentVideoItem()`                                      | `item`                | Returns the current video timeline item. |
| `GetCurrentClipThumbnailImage()`                             | `{thumbnailData}`     | Returns thumbnail data for the current media in the Color Page. |
| `GetTrackName(trackType, trackIndex)`                        | `string`              | Returns the track name for the given `trackType` and `trackIndex`. |
| `SetTrackName(trackType, trackIndex, name)`                  | `Bool`                | Sets the track name for the given `trackType` and `trackIndex`. |
| `DuplicateTimeline(timelineName)`                            | `timeline`            | Duplicates the timeline and returns the created timeline, with the optional `timelineName`. |
| `CreateCompoundClip([timelineItems], {clipInfo})`           | `timelineItem`        | Creates a compound clip from input timeline items with an optional `clipInfo` map and returns the created timeline item. |
| `CreateFusionClip([timelineItems])`                          | `timelineItem`        | Creates a Fusion clip from input timeline items and returns the created timeline item. |
| `ImportIntoTimeline(filePath, {importOptions})`            | `Bool`                | Imports timeline items from an AAF file and optional `importOptions` dict into the timeline, with support for the keys: <br> `autoImportSourceClipsIntoMediaPool`: Bool, specifies if source clips should be imported into media pool, True by default. <br> `ignoreFileExtensionsWhenMatching`: Bool, specifies if file extensions should be ignored when matching, False by default. <br> `linkToSourceCameraFiles`: Bool, specifies if link to source camera files should be enabled, False by default. <br> `useSizingInfo`: Bool, specifies if sizing information should be used, False by default. <br> `importMultiChannelAudioTracksAsLinkedGroups`: Bool, specifies if multi-channel audio tracks should be imported as linked groups, False by default. <br> `insertAdditionalTracks`: Bool, specifies if additional tracks should be inserted, True by default. <br> `insertWithOffset`: string, specifies insert with offset value in timecode format - defaults to "00:00:00:00", applicable if `insertAdditionalTracks` is False. <br> `sourceClipsPath`: string, specifies a filesystem path to search for source clips if the media is inaccessible in their original path and if `ignoreFileExtensionsWhenMatching` is True. <br> `sourceClipsFolders`: string, list of Media Pool folder objects to search for source clips if the media is not present in current folder. |
| `Export(fileName, exportType, exportSubtype)`              | `Bool`                | Exports timeline to `fileName` in the specified `exportType` and `exportSubtype` format. |
| `GetSetting(settingName)`                                   | `string`              | Returns the value of a timeline setting indicated by `settingName`. |
| `SetSetting(settingName, settingValue)`                     | `Bool`                | Sets the timeline setting specified by `settingName` to the given `settingValue`. |
| `InsertGeneratorIntoTimeline(generatorName)`                | `TimelineItem`        | Inserts a generator (indicated by `generatorName`) into the timeline. |
| `InsertFusionGeneratorIntoTimeline(generatorName)`          | `TimelineItem`        | Inserts a Fusion generator (indicated by `generatorName`) into the timeline. |
| `InsertFusionCompositionIntoTimeline()`                     | `TimelineItem`        | Inserts a Fusion composition into the timeline. |
| `InsertOFXGeneratorIntoTimeline(generatorName)`             | `TimelineItem`        | Inserts an OFX generator (indicated by `generatorName`) into the timeline. |
| `InsertTitleIntoTimeline(titleName)`                        | `TimelineItem`        | Inserts a title (indicated by `titleName`) into the timeline. |
| `InsertFusionTitleIntoTimeline(titleName)`                  | `TimelineItem`        | Inserts a Fusion title (indicated by `titleName`) into the timeline. |
| `GrabStill()`                                              | `galleryStill`        | Grabs a still from the current video clip and returns a `GalleryStill` object. |
| `GrabAllStills(stillFrameSource)`                           | `[galleryStill]`      | Grabs stills from all the clips of the timeline at `stillFrameSource` (1 - First frame, 2 - Middle frame). Returns a list of `GalleryStill` objects. |
| `GetUniqueId()`                                             | `string`              | Returns a unique ID for the timeline. |
| `CreateSubtitlesFromAudio({autoCaptionSettings})`           | `Bool`                | Creates subtitles from audio for the timeline using optional `autoCaptionSettings`. Returns True on success, False otherwise. |
| `DetectSceneCuts()`                                         | `Bool`                | Detects and makes scene cuts along the timeline. Returns True if successful, False otherwise. |
| `ConvertTimelineToStereo()`                                 | `Bool`                | Converts timeline to stereo. Returns True if successful, False otherwise. |
| `GetNodeGraph()`                                            | `Graph`               | Returns the timeline's node graph object. |
| `AnalyzeDolbyVision([timelineItems]=[], analysisType=NONE)` | `Bool`                | Analyzes Dolby Vision on clips in the timeline. Returns True if analysis is successful, False otherwise. |
| ~~`GetItemsInTrack(trackType, index)`~~                     | ~~`{items...}`~~      | ~~Returns a dictionary of `TimelineItems` on the specified track (based on `trackType` and `index`).~~ |


### `TimelineItem` Methods

| Method                                    | Return Type            | Comment                                                                                                                                                                                                                                                                                                                 |
|-------------------------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GetName()`                                | `string`               | Returns the item name.                                                                                                                                                                                                                                                                                                  |
| `GetDuration(subframe_precision)`          | `int/float`            | Returns the item duration. Returns fractional frames if `subframe_precision` is `True`.                                                                                                                                                                                                                                 |
| `GetEnd(subframe_precision)`               | `int/float`            | Returns the end frame position on the timeline. Returns fractional frames if `subframe_precision` is `True`.                                                                                                                                                                                                            |
| `GetSourceEndFrame()`                      | `int`                  | Returns the end frame position of the media pool clip in the timeline clip.                                                                                                                                                                                                                                             |
| `GetSourceEndTime()`                       | `float`                | Returns the end time position of the media pool clip in the timeline clip.                                                                                                                                                                                                                                              |
| `GetFusionCompCount()`                     | `int`                  | Returns the number of Fusion compositions associated with the timeline item.                                                                                                                                                                                                                                            |
| `GetFusionCompByIndex(compIndex)`          | `fusionComp`           | Returns the Fusion composition object based on a given index. `1 <= compIndex <= timelineItem.GetFusionCompCount()`.                                                                                                                                                                                                    |
| `GetFusionCompNameList()`                  | `[names...]`           | Returns a list of Fusion composition names associated with the timeline item.                                                                                                                                                                                                                                           |
| `GetFusionCompByName(compName)`            | `fusionComp`           | Returns the Fusion composition object based on a given name.                                                                                                                                                                                                                                                            |
| `GetLeftOffset(subframe_precision)`        | `int/float`            | Returns the maximum extension by frame for clip from the left side. Returns fractional frames if `subframe_precision` is `True`.                                                                                                                                                                                        |
| `GetRightOffset(subframe_precision)`       | `int/float`            | Returns the maximum extension by frame for clip from the right side. Returns fractional frames if `subframe_precision` is `True`.                                                                                                                                                                                       |
| `GetStart(subframe_precision)`             | `int/float`            | Returns the start frame position on the timeline. Returns fractional frames if `subframe_precision` is `True`.                                                                                                                                                                                                          |
| `GetSourceStartFrame()`                    | `int`                  | Returns the start frame position of the media pool clip in the timeline clip.                                                                                                                                                                                                                                           |
| `GetSourceStartTime()`                     | `float`                | Returns the start time position of the media pool clip in the timeline clip.                                                                                                                                                                                                                                            |
| `SetProperty(propertyKey, propertyValue)`   | `Bool`                 | Sets the value of the property "propertyKey" to the value "propertyValue". Refer to "Looking up Timeline item properties" for more information.                                                                                                                                                                        |
| `GetProperty(propertyKey)`                 | `int/[key:value]`      | Returns the value of the specified key. If no key is specified, the method returns a dictionary (Python) or table (Lua) for all supported keys.                                                                                                                                                                        |
| `AddMarker(frameId, color, name, note, duration, customData)` | `Bool` | Creates a new marker at the given `frameId` position with the specified marker information. `customData` is optional and helps to attach user-specific data to the marker.                                                                                                                                              |
| `GetMarkers()`                             | `{markers...}`         | Returns a dictionary (`frameId -> {information}`) of all markers and dictionaries with their information. Example: `{96.0: {'color': 'Green', 'duration': 1.0, 'note': '', 'name': 'Marker 1', 'customData': ''}}` indicates a single green marker at clip offset `96`.                                             |
| `GetMarkerByCustomData(customData)`        | `{markers...}`         | Returns marker `{information}` for the first matching marker with the specified `customData`.                                                                                                                                                                                                                           |
| `UpdateMarkerCustomData(frameId, customData)` | `Bool`              | Updates `customData` (string) for the marker at the given `frameId` position. `customData` is not exposed via UI and is useful for scripting developers to attach user-specific data to markers.                                                                                                                      |
| `GetMarkerCustomData(frameId)`             | `string`               | Returns the `customData` string for the marker at the given `frameId` position.                                                                                                                                                                                                                                         |
| `DeleteMarkersByColor(color)`              | `Bool`                 | Deletes all markers of the specified color from the timeline item. `"All"` as an argument deletes all color markers.                                                                                                                                                                                                   |
| `DeleteMarkerAtFrame(frameNum)`            | `Bool`                 | Deletes the marker at the specified frame number from the timeline item.                                                                                                                                                                                                                                                |
| `DeleteMarkerByCustomData(customData)`     | `Bool`                 | Deletes the first matching marker with the specified `customData`.                                                                                                                                                                                                                                                      |
| `AddFlag(color)`                           | `Bool`                 | Adds a flag with the specified color (string).                                                                                                                                                                                                                                                                          |
| `GetFlagList()`                            | `[colors...]`          | Returns a list of flag colors assigned to the item.                                                                                                                                                                                                                                                                     |
| `ClearFlags(color)`                        | `Bool`                 | Clears flags of the specified color. An `"All"` argument is supported to clear all flags.                                                                                                                                                                                                                               |
| `GetClipColor()`                           | `string`               | Returns the item color as a string.                                                                                                                                                                                                                                                                                     |
| `SetClipColor(colorName)`                  | `Bool`                 | Sets the item color based on the `colorName` (string).                                                                                                                                                                                                                                                                  |
| `ClearClipColor()`                         | `Bool`                 | Clears the item color.                                                                                                                                                                                                                                                                                                  |
| `AddFusionComp()`                          | `fusionComp`           | Adds a new Fusion composition associated with the timeline item.                                                                                                                                                                                                                                                        |
| `ImportFusionComp(path)`                   | `fusionComp`           | Imports a Fusion composition from the specified file path by creating and adding a new composition for the item.                                                                                                                                                                                                        |
| `ExportFusionComp(path, compIndex)`        | `Bool`                 | Exports the Fusion composition based on the specified index to the specified path.                                                                                                                                                                                                                                      |
| `DeleteFusionCompByName(compName)`         | `Bool`                 | Deletes the named Fusion composition.                                                                                                                                                                                                                                                                                   |
| `LoadFusionCompByName(compName)`           | `fusionComp`           | Loads the named Fusion composition as the active composition.                                                                                                                                                                                                                                                           |
| `RenameFusionCompByName(oldName, newName)` | `Bool`                 | Renames the Fusion composition identified by `oldName`.                                                                                                                                                                                                                                                                 |
| `AddVersion(versionName, versionType)`     | `Bool`                 | Adds a new color version for a video clip based on `versionType` (`0` - local, `1` - remote).                                                                                                                                                                                                                           |
| `GetCurrentVersion()`                      | `{versionName...}`     | Returns the current version of the video clip. The returned value will have the keys `versionName` and `versionType` (`0` - local, `1` - remote).                                                                                                                                                                      |
| `DeleteVersionByName(versionName, versionType)` | `Bool`             | Deletes a color version by name and `versionType` (`0` - local, `1` - remote).                                                                                                                                                                                                                                          |
| `LoadVersionByName(versionName, versionType)` | `Bool`             | Loads a named color version as the active version. `versionType`: `0` - local, `1` - remote.                                                                                                                                                                                                                            |
| `RenameVersionByName(oldName, newName, versionType)` | `Bool`         | Renames the color version identified by `oldName` and `versionType` (`0` - local, `1` - remote).                                                                                                                                                                                                                        |
| `GetVersionNameList(versionType)`          | `[names...]`           | Returns a list of all color versions for the specified `versionType` (`0` - local, `1` - remote).                                                                                                                                                                                                                       |
| `GetMediaPoolItem()`                       | `MediaPoolItem`        | Returns the media pool item corresponding to the timeline item, if one exists.                                                                                                                                                                                                                                          |
| `GetStereoConvergenceValues()`             | `{keyframes...}`       | Returns a dictionary (`offset -> value`) of keyframe offsets and respective convergence values.                                                                                                                                                                                                                         |
| `GetStereoLeftFloatingWindowParams()`      | `{keyframes...}`       | For the left eye, returns a dictionary (`offset -> dict`) of keyframe offsets and respective floating window params. The value at a particular offset includes the left, right, top, and bottom floating window values.                                                                                                |
| `GetStereoRightFloatingWindowParams()`     | `{keyframes...}`       | For the right eye, returns a dictionary (`offset -> dict`) of keyframe offsets and respective floating window params. The value at a particular offset includes the left, right, top, and bottom floating window values.                                                                                               |
| `ApplyArriCdlLut()`                        | `Bool`                 | Applies ARRI CDL and LUT. Returns `True` if successful, `False` otherwise.                                                                                                                                                                                                                                              |
| `SetCDL([CDL map])`                        | `Bool`                 | Sets CDL values using a map with keys: `"NodeIndex"`, `"Slope"`, `"Offset"`, `"Power"`, and `"Saturation"`. Example: `SetCDL({"NodeIndex": "1", "Slope": "0.5 0.4 0.2", "Offset": "0.4 0.3 0.2", "Power": "0.6 0.7 0.8", "Saturation": "0.65"})`.                                                                         |
| `AddTake(mediaPoolItem, startFrame, endFrame)` | `Bool`             | Adds `mediaPoolItem` as a new take. Initializes a take selector for the timeline item if needed. By default, the full clip extents are added. `startFrame` (int) and `endFrame` (int) are optional arguments to specify the extents.                                                                                   |
| `GetSelectedTakeIndex()`                   | `int`                  | Returns the index of the currently selected take, or `0` if the clip is not a take selector.                                                                                                                                                                                                                            |
| `GetTakesCount()`                          | `int`                  | Returns the number of takes in the take selector, or `0` if the clip is not a take selector.                                                                                                                                                                                                                            |
| `GetTakeByIndex(idx)`                      | `{takeInfo...}`        | Returns a dictionary (keys `"startFrame"`, `"endFrame"`, and `"mediaPoolItem"`) with take info for the specified index.                                                                                                                                                                                                |
| `DeleteTakeByIndex(idx)`                   | `Bool`                 | Deletes a take by index, where `1 <= idx <= number of takes`.                                                                                                                                                                                                                                                           |
| `SelectTakeByIndex(idx)`                   | `Bool`                 | Selects a take by index, where `1 <= idx <= number of takes`.                                                                                                                                                                                                                                                           |
| `FinalizeTake()`                           | `Bool`                 | Finalizes take selection.                                                                                                                                                                                                                                                                                               |
| `CopyGrades([tgtTimelineItems])`           | `Bool`                 | Copies the current node stack layer grade to the same layer for each item in `tgtTimelineItems`. Returns `True` if successful.                                                                                                                                                                                          |
| `SetClipEnabled(Bool)`                     | `Bool`                 | Sets the clip enabled status based on the argument.                                                                                                                                                                                                                                                                     |
| `GetClipEnabled()`                         | `Bool`                 | Gets the clip enabled status.                                                                                                                                                                                                                                                                                           |
| `UpdateSidecar()`                          | `Bool`                 | Updates the sidecar file for BRAW clips or the RMD file for R3D clips.                                                                                                                                                                                                                                                 |
| `GetUniqueId()`                            | `string`               | Returns a unique ID for the timeline item.                                                                                                                                                                                                                                                                              |
| `LoadBurnInPreset(presetName)`             | `Bool`                 | Loads a user-defined data burn-in preset for the clip with the specified `presetName` (string). Returns `True` if successful.                                                                                                                                                                                           |
| `CreateMagicMask(mode)`                    | `Bool`                 | Returns `True` if the magic mask was created successfully, `False` otherwise. `mode` can be `"F"` (forward), `"B"` (backward), or `"BI"` (bidirectional).                                                                                                                                                              |
| `RegenerateMagicMask()`                    | `Bool`                 | Returns `True` if the magic mask was regenerated successfully, `False` otherwise.                                                                                                                                                                                                                                       |
| `Stabilize()`                              | `Bool`                 | Returns `True` if stabilization was successful, `False` otherwise.                                                                                                                                                                                                                                                      |
| `SmartReframe()`                           | `Bool`                 | Performs Smart Reframe. Returns `True` if successful, `False` otherwise.                                                                                                                                                                                                                                                |
| `GetNodeGraph(layerIdx)`                   | `Graph`                | Returns the clip's node graph object at `layerIdx` (int, optional). Returns the first layer if `layerIdx` is skipped. `1 <= layerIdx <= project.GetSetting("nodeStackLayers")`.                                                                                                                                        |
| `GetColorGroup()`                          | `ColorGroup`           | Returns the clip's color group, if one exists.                                                                                                                                                                                                                                                                          |
| `AssignToColorGroup(ColorGroup)`           | `Bool`                 | Returns `True` if the TimelineItem was successfully assigned to the given `ColorGroup`. `ColorGroup` must be an existing group in the current project.                                                                                                                                                                 |
| `RemoveFromColorGroup()`                   | `Bool`                 | Returns `True` if the TimelineItem was successfully removed from the `ColorGroup` it is in.                                                                                                                                                                                                                             |
| `ExportLUT(exportType, path)`              | `Bool`                 | Exports LUTs from the TimelineItem based on the value passed in `exportType` (enum) for LUT size. Saves the generated LUT in the provided `path` (string), which should include the intended file name. If an empty or incorrect extension is provided, the appropriate extension (.cube/.vlt) will be appended.    |
| `GetLinkedItems()`                         | `[TimelineItems]`      | Returns a list of linked timeline items.                                                                                                                                                                                                                                                                                |
| `GetTrackTypeAndIndex()`                   | `[trackType, trackIndex]` | Returns a list of two values that correspond to the TimelineItem's `trackType` (string) and `trackIndex` (int) respectively. `trackType` is one of `"audio"`, `"video"`, `"subtitle"`, and `1 <= trackIndex <= Timeline.GetTrackCount(trackType)`.                               |
| `GetSourceAudioChannelMapping()`           | `json formatted string` | Returns a string with the TimelineItem's audio mapping information.                                                                                                                                                                                                                                                    |
| ~~`GetFusionCompNames()`~~            | ~~`{names...}`~~         | ~~Returns a dictionary of Fusion composition names associated with the timeline item.~~           |
| ~~`GetFlags()`~~                      | ~~`{colors...}`~~        | ~~Returns a dictionary of flag colors assigned to the item.~~                                     |
| ~~`GetVersionNames(versionType)`~~    | ~~`{names...}`~~         | ~~Returns a dictionary of version names by the specified `versionType` (0 for local, 1 for remote).~~ |
| ~~`GetNumNodes()`~~                   | ~~`int`~~                | ~~Returns the number of nodes in the current graph for the timeline item.~~                       |
| ~~`SetLUT(nodeIndex, lutPath)`~~      | ~~`Bool`~~               | ~~Sets the LUT on the specified node by `nodeIndex`. `lutPath` can be absolute or relative. The operation is successful if the LUT path is valid.~~ |
| ~~`GetLUT(nodeIndex)`~~               | ~~`string`~~             | ~~Gets the relative LUT path based on the specified `nodeIndex`.~~                                |
| ~~`GetNodeLabel(nodeIndex)`~~         | ~~`string`~~             | ~~Returns the label of the node at `nodeIndex`.~~                                                 |


### `Gallery` Methods

| Method                                    | Return Type            | Comment                                                                                                                                                 |
|-------------------------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GetAlbumName(galleryStillAlbum)`          | `string`               | Returns the name of the specified `GalleryStillAlbum` object `galleryStillAlbum`.                                                                       |
| `SetAlbumName(galleryStillAlbum, albumName)` | `Bool`              | Sets the name of the specified `GalleryStillAlbum` object `galleryStillAlbum` to the new name `albumName`.                                              |
| `GetCurrentStillAlbum()`                   | `GalleryStillAlbum`    | Returns the current album as a `GalleryStillAlbum` object.                                                                                              |
| `SetCurrentStillAlbum(galleryStillAlbum)`   | `Bool`                 | Sets the current album to the specified `GalleryStillAlbum` object `galleryStillAlbum`.                                                                 |
| `GetGalleryStillAlbums()`                  | `[GalleryStillAlbum]`  | Returns a list of all gallery albums as `GalleryStillAlbum` objects.                                                                                    |


### `GalleryStillAlbum` Methods

| Method                                           | Return Type        | Comment                                                                                                                                                                                                                     |
|--------------------------------------------------|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GetStills()`                                    | `[GalleryStill]`   | Returns the list of `GalleryStill` objects in the album.                                                                                                                                                                   |
| `GetLabel(galleryStill)`                         | `string`           | Returns the label of the specified `galleryStill`.                                                                                                                                                                         |
| `SetLabel(galleryStill, label)`                  | `Bool`             | Sets the new `label` for the specified `GalleryStill` object `galleryStill`.                                                                                                                                               |
| `ImportStills([filePaths])`                      | `Bool`             | Imports a `GalleryStill` from each `filePath` in the `filePaths` list. Returns `True` if at least one still is imported successfully, `False` otherwise.                                                                   |
| `ExportStills([galleryStill], folderPath, filePrefix, format)` | `Bool` | Exports the specified list of `GalleryStill` objects `[galleryStill]` to the directory `folderPath`, with filename prefix `filePrefix`, using file format `format` (supported formats: dpx, cin, tif, jpg, png, ppm, bmp, xpm, drx). |
| `DeleteStills([galleryStill])`                   | `Bool`             | Deletes the specified list of `GalleryStill` objects `[galleryStill]`.                                                                                                                                                     |

### `GalleryStill` Methods

This class does not provide any API functions but the object type is used by functions in other classes.

### `Graph` Methods

| Method                                      | Return Type          | Comment                                                                                                                                                                                                                                         |
|---------------------------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GetNumNodes()`                             | `int`                | Returns the number of nodes in the graph.                                                                                                                                                                                                      |
| `SetLUT(nodeIndex, lutPath)`                | `Bool`               | Sets the LUT on the node at the specified `nodeIndex`. Valid indices are `1 <= nodeIndex <= self.GetNumNodes()`. `lutPath` can be an absolute or relative path (relative paths are based on custom or master LUT paths discovered by Resolve). |
| `GetLUT(nodeIndex)`                         | `string`             | Gets the relative LUT path for the node at the specified `nodeIndex`, where `1 <= nodeIndex <= total number of nodes`.                                                                                                                         |
| `GetNodeLabel(nodeIndex)`                   | `string`             | Returns the label of the node at the specified `nodeIndex`.                                                                                                                                                                                    |
| `GetToolsInNode(nodeIndex)`                 | `[string]`           | Returns a list of tools used in the node specified by `nodeIndex`.                                                                                                                                                                             |
| `SetNodeEnabled(nodeIndex, isEnabled)`      | `Bool`               | Enables or disables the node at the specified `nodeIndex` based on the `isEnabled` boolean value. Valid indices are `1 <= nodeIndex <= self.GetNumNodes()`.                                                                                    |


### `ColorGroup` Methods

| Method                                    | Return Type          | Comment                                                                                                                       |
|-------------------------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------|
| `GetName()`                                | `string`             | Returns the name of the `ColorGroup`.                                                                                         |
| `SetName(groupName)`                       | `Bool`               | Renames the `ColorGroup` to the specified `groupName`.                                                                        |
| `GetClipsInTimeline(Timeline=CurrTimeline)` | `[TimelineItem]`    | Returns a list of `TimelineItem` objects in the `ColorGroup` within the specified `Timeline`. Defaults to the current timeline. |
| `GetPreClipNodeGraph()`                    | `Graph`              | Returns the `ColorGroup` Pre-clip node graph.                                                                                 |
| `GetPostClipNodeGraph()`                   | `Graph`              | Returns the `ColorGroup` Post-clip node graph.                                                                                |


List and Dict Data Structures
-----------------------------
Beside primitive data types, Resolve's Python API mainly uses list and dict data structures. Lists are denoted by [ ... ] and dicts are denoted by { ... } above.
As Lua does not support list and dict data structures, the Lua API implements "list" as a table with indices, e.g. `{ [1] = listValue1, [2] = listValue2, ... }`.
Similarly the Lua API implements "dict" as a table with the dictionary key as first element, e.g. `{ [dictKey1] = dictValue1, [dictKey2] = dictValue2, ... }`.

Keyframe Mode information
-------------------------
This section covers additional notes for the functions `Resolve.GetKeyframeMode()` and `Resolve.SetKeyframeMode(keyframeMode)`.

`keyframeMode` can be one of the following enums:
* `resolve.KEYFRAME_MODE_ALL     == 0`
* `resolve.KEYFRAME_MODE_COLOR   == 1`
* `resolve.KEYFRAME_MODE_SIZING  == 2`

Integer values returned by `Resolve.GetKeyframeMode()` will correspond to the enums above.

Cloud Projects Settings
--------------------------------------
This section covers additional notes for the functions "ProjectManager:CreateCloudProject," "ProjectManager:ImportCloudProject," and "ProjectManager:RestoreCloudProject"

All three functions take in a {cloudSettings} dict, that have the following keys:
* `resolve.CLOUD_SETTING_PROJECT_NAME`: `String`, [`""` by default]
* `resolve.CLOUD_SETTING_PROJECT_MEDIA_PATH`: `String`, [`""` by default]
* `resolve.CLOUD_SETTING_IS_COLLAB`: `Bool`, [`False` by default]
* `resolve.CLOUD_SETTING_SYNC_MODE`: `syncMode` (see below), [`resolve.CLOUD_SYNC_PROXY_ONLY` by default]
* `resolve.CLOUD_SETTING_IS_CAMERA_ACCESS`: `Bool` [`False` by default]

Where `syncMode` is one of the following values:
* `resolve.CLOUD_SYNC_NONE`,
* `resolve.CLOUD_SYNC_PROXY_ONLY`,
* `resolve.CLOUD_SYNC_PROXY_AND_ORIG`

All three `ProjectManager:CreateCloudProject`, `ProjectManager:ImportCloudProject`, and `ProjectManager:RestoreCloudProject` require `resolve.PROJECT_MEDIA_PATH` to be defined. `ProjectManager:CreateCloudProject` also requires `resolve.PROJECT_NAME` to be defined.

Looking up Project and Clip properties
--------------------------------------
This section covers additional notes for the functions `Project:GetSetting`, `Project:SetSetting`, `Timeline:GetSetting`, `Timeline:SetSetting`, `MediaPoolItem:GetClipProperty` and
`MediaPoolItem:SetClipProperty`. These functions are used to get and set properties otherwise available to the user through the Project Settings and the Clip Attributes dialogs.

The functions follow a key-value pair format, where each property is identified by a key (the `settingName` or `propertyName` parameter) and possesses a value (typically a text value). Keys and values are
designed to be easily correlated with parameter names and values in the Resolve UI. Explicitly enumerated values for some parameters are listed below.

Some properties may be read only - these include intrinsic clip properties like date created or sample rate, and properties that can be disabled in specific application contexts (e.g., custom colorspaces
in an ACES workflow, or output sizing parameters when behavior is set to match timeline).

### Getting values:
Invoke `Project:GetSetting`, `Timeline:GetSetting` or `MediaPoolItem:GetClipProperty` with the appropriate property key. To get a snapshot of all queryable properties (keys and values), you can call
`Project:GetSetting`, `Timeline:GetSetting` or `MediaPoolItem:GetClipProperty` without parameters (or with a `NoneType` or a blank property key). Using specific keys to query individual properties will
be faster. Note that getting a property using an invalid key will return a trivial result.

### Setting values:
Invoke `Project:SetSetting`, `Timeline:SetSetting` or `MediaPoolItem:SetClipProperty` with the appropriate property key and a valid value. When setting a parameter, please check the return value to
ensure the success of the operation. You can troubleshoot the validity of keys and values by setting the desired result from the UI and checking property snapshots before and after the change.

### The following Project properties have specifically enumerated values:
- `superScale` - the property value is an enumerated integer between `0` and `4` with these meanings: `0=Auto`, `1=no scaling`, and `2`, `3`, and `4` represent the Super Scale multipliers `2x`, `3x`, and `4x`.
    - for super scale multiplier `'2x Enhanced'`, exactly `4` arguments must be passed as outlined below. If less than `4` arguments are passed, it will default to `2x`.
  
**Affects:**
- `x = Project:GetSetting('superScale')` and `Project:SetSetting('superScale', x)`
- for `'2x Enhanced'` --> `Project:SetSetting('superScale', 2, sharpnessValue, noiseReductionValue)`, where `sharpnessValue` is a float in the range `[0.0, 1.0]` and `noiseReductionValue` is a float in the range `[0.0, 1.0]`

- `"timelineFrameRate"` - the property value is one of the frame rates available to the user in project settings under "Timeline frame rate" option. Drop Frame can be configured for supported frame rates
  by appending the frame rate with `"DF"`, e.g., `"29.97 DF"` will enable drop frame and `"29.97"` will disable drop frame

**Affects:**
- `x = Project:GetSetting('timelineFrameRate')` and `Project:SetSetting('timelineFrameRate', x)`

### The following Clip properties have specifically enumerated values:
- `"Super Scale"` - the property value is an enumerated integer between `1` and `4` with these meanings: `1=no scaling`, and `2`, `3`, and `4` represent the Super Scale multipliers `2x`, `3x`, and `4x`.
    - for super scale multiplier `'2x Enhanced'`, exactly `4` arguments must be passed as outlined below. If less than `4` arguments are passed, it will default to `2x`.
  
**Affects:**
- `x = MediaPoolItem:GetClipProperty('Super Scale')` and `MediaPoolItem:SetClipProperty('Super Scale', x)`
- for `'2x Enhanced'` --> `MediaPoolItem:SetClipProperty('Super Scale', 2, sharpnessValue, noiseReductionValue)`, where `sharpnessValue` is a float in the range `[0.0, 1.0]` and `noiseReductionValue` is a float in the range `[0.0, 1.0]`

- `Cloud Sync` - the property value is an enumerated integer that will correspond to one of the following enums:
    - `resolve.CLOUD_SYNC_DEFAULT                == -1`
    - `resolve.CLOUD_SYNC_DOWNLOAD_IN_QUEUE      == 0`
    - `resolve.CLOUD_SYNC_DOWNLOAD_IN_PROGRESS   == 1`
    - `resolve.CLOUD_SYNC_DOWNLOAD_SUCCESS       == 2`
    - `resolve.CLOUD_SYNC_DOWNLOAD_FAIL          == 3`
    - `resolve.CLOUD_SYNC_DOWNLOAD_NOT_FOUND     == 4`
    - `resolve.CLOUD_SYNC_UPLOAD_IN_QUEUE        == 5`
    - `resolve.CLOUD_SYNC_UPLOAD_IN_PROGRESS     == 6`
    - `resolve.CLOUD_SYNC_UPLOAD_SUCCESS         == 7`
    - `resolve.CLOUD_SYNC_UPLOAD_FAIL            == 8`
    - `resolve.CLOUD_SYNC_UPLOAD_NOT_FOUND       == 9`
    - `resolve.CLOUD_SYNC_SUCCESS                == 10`

Audio Mapping
---------------
This section covers the output for `mpItem.GetAudioMapping()` and `timelineItem.GetSourceAudioChannelMapping()`
Mapping format (json result) is similar for `mpItem` and `timelineItem`.

This section will follow an example of an `mpItem` that has audio from its embedded source, and from two other clips that are linked to it.
The audio clip attributes of this `mpItem` will show 3 tracks.

Assume that<br>
(A) the embedded track is of format/type 'stereo' (2 channels),<br>
(B) linked clip 1 track is of format/type '7.1' (8 channels),<br>
(C) linked clip 2 track is '5.1' (6 channels)<br>
and assume that the format/type was not changed further.

`mpItem.GetAudioMapping()` returns a string of the form:
```
    {
      "embedded_audio_channels": 2,                 # Total number of embedded channels across all tracks
      "linked_audio": {                             # A list of only linked audio information
        "1": {                                      # Same as (B) above
          "channels": 8,
          "path": FILE_PATH
        },
        "2": {                                      # Same as (C) above
          "channels": 6,
          "path": FILE_PATH
        }
      },
      "track_mapping": {                            # Listing of all the tracks. Output here will match what is seen in the audio clip attributes menu on the UI.
        "1": {
          "channel_idx": [1, 3],                    # In this case, channel index '1' corresponds to first channel of (A), channel index '3' will correspond to the first channel of (B)
          "mute": true,                             # Mute 'true' indicates track is muted. Valid value is true/false.
          "type": "Stereo"                          # The length of the 'channel_idx' list will always correspond to the number of channels the format specified in 'type' will allow.
                                                    # In this case, 'Stereo' allows 2 channels and so the length of the 'channel_idx' list is 2.
        },
        "2": {
          "channel_idx": [3, 4, 5, 6, 7, 8, 9, 10], # Channel indices here are following the default for (B)
          "mute": true,
          "type": "7.1"
        },
        "3": {
          "channel_idx": [1, 1, 1, 1, 15, 16],      # The first four channels for this track correspond to the first channel of (A), and the final 2 follow the default for (C)
          "mute": false,
          "type": "5.1"
        }
      }
    }
```

Auto Caption Settings
----------------------
This section covers the supported settings for the method `Timeline.CreateSubtitlesFromAudio({autoCaptionSettings})`

The parameter setting is a dictionary containing the following keys:
* `resolve.SUBTITLE_LANGUAGE`: `languageID` (see below), [`resolve.AUTO_CAPTION_AUTO` by default]
* `resolve.SUBTITLE_CAPTION_PRESET`: `presetType` (see below), [`resolve.AUTO_CAPTION_SUBTITLE_DEFAULT` by default]
* `resolve.SUBTITLE_CHARS_PER_LINE`: `Number` between 1 and 60 inclusive [`42` by default]
* `resolve.SUBTITLE_LINE_BREAK`: `lineBreakType` (see below), [`resolve.AUTO_CAPTION_LINE_SINGLE` by default]
* `resolve.SUBTITLE_GAP`: `Number` between 0 and 10 inclusive [`0` by default]

Note that the default values for some keys may change based on values defined for other keys, as per the UI.
For example, if the following dictionary is supplied,
    `CreateSubtitlesFromAudio( { resolve.SUBTITLE_LANGUAGE = resolve.AUTO_CAPTION_KOREAN,
                                resolve.SUBTITLE_CAPTION_PRESET = resolve.AUTO_CAPTION_NETFLIX } )`
the default value for `resolve.SUBTITLE_CHARS_PER_LINE` will be 16 instead of 42

`languageID`s:
* `resolve.AUTO_CAPTION_AUTO`
* `resolve.AUTO_CAPTION_DANISH`
* `resolve.AUTO_CAPTION_DUTCH`
* `resolve.AUTO_CAPTION_ENGLISH`
* `resolve.AUTO_CAPTION_FRENCH`
* `resolve.AUTO_CAPTION_GERMAN`
* `resolve.AUTO_CAPTION_ITALIAN`
* `resolve.AUTO_CAPTION_JAPANESE`
* `resolve.AUTO_CAPTION_KOREAN`
* `resolve.AUTO_CAPTION_MANDARIN_SIMPLIFIED`
* `resolve.AUTO_CAPTION_MANDARIN_TRADITIONAL`
* `resolve.AUTO_CAPTION_NORWEGIAN`
* `resolve.AUTO_CAPTION_PORTUGUESE`
* `resolve.AUTO_CAPTION_RUSSIAN`
* `resolve.AUTO_CAPTION_SPANISH`
* `resolve.AUTO_CAPTION_SWEDISH`

`presetType`s:
* `resolve.AUTO_CAPTION_SUBTITLE_DEFAULT`
* `resolve.AUTO_CAPTION_TELETEXT`
* `resolve.AUTO_CAPTION_NETFLIX`

`lineBreakType`s:
* `resolve.AUTO_CAPTION_LINE_SINGLE`
* `resolve.AUTO_CAPTION_LINE_DOUBLE`


Looking up Render Settings
--------------------------
This section covers the supported settings for the method `SetRenderSettings({settings})`

The parameter `settings` is a dictionary containing the following keys:
- `SelectAllFrames`: Bool (when set `True`, the settings `MarkIn` and `MarkOut` are ignored)
- `MarkIn`: int
- `MarkOut`: int
- `TargetDir`: string
- `CustomName`: string
- `UniqueFilenameStyle`: `0` - Prefix, `1` - Suffix.
- `ExportVideo`: Bool
- `ExportAudio`: Bool
- `FormatWidth`: int
- `FormatHeight`: int
- `FrameRate`: float (examples: `23.976`, `24`)
- `PixelAspectRatio`: string (for SD resolution: `"16_9"` or `"4_3"`) (other resolutions: `"square"` or `"cinemascope"`)
- `VideoQuality` possible values for current codec (if applicable):
    - `0` (int) - will set quality to automatic
    - `[1 -> MAX]` (int) - will set input bit rate
    - `["Least", "Low", "Medium", "High", "Best"]` (String) - will set input quality level
- `AudioCodec`: string (example: `"aac"`)
- `AudioBitDepth`: int
- `AudioSampleRate`: int
- `ColorSpaceTag`: string (example: `"Same as Project"`, `"AstroDesign"`)
- `GammaTag`: string (example: `"Same as Project"`, `"ACEScct"`)
- `ExportAlpha`: Bool
- `EncodingProfile`: string (example: `"Main10"`). Can only be set for `H.264` and `H.265`.
- `MultiPassEncode`: Bool. Can only be set for `H.264`.
- `AlphaMode`: `0` - Premultiplied, `1` - Straight. Can only be set if `ExportAlpha` is true.
- `NetworkOptimization`: Bool. Only supported by `QuickTime` and `MP4` formats.


Looking up timeline export properties
-------------------------------------
This section covers the parameters for the argument `Export(fileName, exportType, exportSubtype)`.

`exportType` can be one of the following constants:
- `resolve.EXPORT_AAF`
- `resolve.EXPORT_DRT`
- `resolve.EXPORT_EDL`
- `resolve.EXPORT_FCP_7_XML`
- `resolve.EXPORT_FCPXML_1_8`
- `resolve.EXPORT_FCPXML_1_9`
- `resolve.EXPORT_FCPXML_1_10`
- `resolve.EXPORT_HDR_10_PROFILE_A`
- `resolve.EXPORT_HDR_10_PROFILE_B`
- `resolve.EXPORT_TEXT_CSV`
- `resolve.EXPORT_TEXT_TAB`
- `resolve.EXPORT_DOLBY_VISION_VER_2_9`
- `resolve.EXPORT_DOLBY_VISION_VER_4_0`
- `resolve.EXPORT_DOLBY_VISION_VER_5_1`
- `resolve.EXPORT_OTIO`
- `resolve.EXPORT_ALE`
- `resolve.EXPORT_ALE_CDL`

`exportSubtype` can be one of the following enums:
- `resolve.EXPORT_NONE`
- `resolve.EXPORT_AAF_NEW`
- `resolve.EXPORT_AAF_EXISTING`
- `resolve.EXPORT_CDL`
- `resolve.EXPORT_SDL`
- `resolve.EXPORT_MISSING_CLIPS`

Please note that `exportSubtype` is a required parameter for `resolve.EXPORT_AAF` and `resolve.EXPORT_EDL`. For the rest of the `exportType`, `exportSubtype` is ignored.

When `exportType` is `resolve.EXPORT_AAF`, valid `exportSubtype` values are `resolve.EXPORT_AAF_NEW` and `resolve.EXPORT_AAF_EXISTING`.

When `exportType` is `resolve.EXPORT_EDL`, valid `exportSubtype` values are `resolve.EXPORT_CDL`, `resolve.EXPORT_SDL`, `resolve.EXPORT_MISSING_CLIPS`, and `resolve.EXPORT_NONE`.

Note: Replace `'resolve.'` when using the constants above if a different Resolve class instance name is used.


Unsupported exportType types
---------------------------------
Starting with DaVinci Resolve 18.1, the following export types are not supported:
- `resolve.EXPORT_FCPXML_1_3`
- `resolve.EXPORT_FCPXML_1_4`
- `resolve.EXPORT_FCPXML_1_5`
- `resolve.EXPORT_FCPXML_1_6`
- `resolve.EXPORT_FCPXML_1_7`


Looking up Timeline item properties
-----------------------------------
This section covers additional notes for the function `TimelineItem:SetProperty` and `TimelineItem:GetProperty`. These functions are used to get and set properties mentioned.

The supported keys with their accepted values are:
- `Pan`: floating point values from `-4.0*width` to `4.0*width`
- `Tilt`: floating point values from `-4.0*height` to `4.0*height`
- `ZoomX`: floating point values from `0.0` to `100.0`
- `ZoomY`: floating point values from `0.0` to `100.0`
- `ZoomGang`: a boolean value
- `RotationAngle`: floating point values from `-360.0` to `360.0`
- `AnchorPointX`: floating point values from `-4.0*width` to `4.0*width`
- `AnchorPointY`: floating point values from `-4.0*height` to `4.0*height`
- `Pitch`: floating point values from `-1.5` to `1.5`
- `Yaw`: floating point values from `-1.5` to `1.5`
- `FlipX`: boolean value for flipping horizontally
- `FlipY`: boolean value for flipping vertically
- `CropLeft`: floating point values from `0.0` to `width`
- `CropRight`: floating point values from `0.0` to `width`
- `CropTop`: floating point values from `0.0` to `height`
- `CropBottom`: floating point values from `0.0` to `height`
- `CropSoftness`: floating point values from `-100.0` to `100.0`
- `CropRetain`: boolean value for "Retain Image Position" checkbox
- `DynamicZoomEase`: A value from the following constants:
  - `DYNAMIC_ZOOM_EASE_LINEAR = 0`
  - `DYNAMIC_ZOOM_EASE_IN`
  - `DYNAMIC_ZOOM_EASE_OUT`
  - `DYNAMIC_ZOOM_EASE_IN_AND_OUT`
- `CompositeMode`: A value from the following constants:
  - `COMPOSITE_NORMAL = 0`
  - `COMPOSITE_ADD`
  - `COMPOSITE_SUBTRACT`
  - `COMPOSITE_DIFF`
  - `COMPOSITE_MULTIPLY`
  - `COMPOSITE_SCREEN`
  - `COMPOSITE_OVERLAY`
  - `COMPOSITE_HARDLIGHT`
  - `COMPOSITE_SOFTLIGHT`
  - `COMPOSITE_DARKEN`
  - `COMPOSITE_LIGHTEN`
  - `COMPOSITE_COLOR_DODGE`
  - `COMPOSITE_COLOR_BURN`
  - `COMPOSITE_EXCLUSION`
  - `COMPOSITE_HUE`
  - `COMPOSITE_SATURATE`
  - `COMPOSITE_COLORIZE`
  - `COMPOSITE_LUMA_MASK`
  - `COMPOSITE_DIVIDE`
  - `COMPOSITE_LINEAR_DODGE`
  - `COMPOSITE_LINEAR_BURN`
  - `COMPOSITE_LINEAR_LIGHT`
  - `COMPOSITE_VIVID_LIGHT`
  - `COMPOSITE_PIN_LIGHT`
  - `COMPOSITE_HARD_MIX`
  - `COMPOSITE_LIGHTER_COLOR`
  - `COMPOSITE_DARKER_COLOR`
  - `COMPOSITE_FOREGROUND`
  - `COMPOSITE_ALPHA`
  - `COMPOSITE_INVERTED_ALPHA`
  - `COMPOSITE_LUM`
  - `COMPOSITE_INVERTED_LUM`
- `Opacity`: floating point value from `0.0` to `100.0`
- `Distortion`: floating point value from `-1.0` to `1.0`
- `RetimeProcess`: A value from the following constants:
  - `RETIME_USE_PROJECT = 0`
  - `RETIME_NEAREST`
  - `RETIME_FRAME_BLEND`
  - `RETIME_OPTICAL_FLOW`
- `MotionEstimation`: A value from the following constants:
  - `MOTION_EST_USE_PROJECT = 0`
  - `MOTION_EST_STANDARD_FASTER`
  - `MOTION_EST_STANDARD_BETTER`
  - `MOTION_EST_ENHANCED_FASTER`
  - `MOTION_EST_ENHANCED_BETTER`
  - `MOTION_EST_SPEED_WARP_BETTER`
  - `MOTION_EST_SPEED_WARP_FASTER`
- `Scaling`: A value from the following constants:
  - `SCALE_USE_PROJECT = 0`
  - `SCALE_CROP`
  - `SCALE_FIT`
  - `SCALE_FILL`
  - `SCALE_STRETCH`
- `ResizeFilter`: A value from the following constants:
  - `RESIZE_FILTER_USE_PROJECT = 0`
  - `RESIZE_FILTER_SHARPER`
  - `RESIZE_FILTER_SMOOTHER`
  - `RESIZE_FILTER_BICUBIC`
  - `RESIZE_FILTER_BILINEAR`
  - `RESIZE_FILTER_BESSEL`
  - `RESIZE_FILTER_BOX`
  - `RESIZE_FILTER_CATMULL_ROM`
  - `RESIZE_FILTER_CUBIC`
  - `RESIZE_FILTER_GAUSSIAN`
  - `RESIZE_FILTER_LANCZOS`
  - `RESIZE_FILTER_MITCHELL`
  - `RESIZE_FILTER_NEAREST_NEIGHBOR`
  - `RESIZE_FILTER_QUADRATIC`
  - `RESIZE_FILTER_SINC`
  - `RESIZE_FILTER_LINEAR`
Values beyond the range will be clipped
width and height are same as the UI max limits

The arguments can be passed as a key and value pair or they can be grouped together into a dictionary (for python) or table (for lua) and passed
as a single argument.

Getting the values for the keys that uses constants will return the number which is in the constant

ExportLUT notes
---------------
The following section covers additional notes for `TimelineItem.ExportLUT(exportType, path)`.

Supported values for `exportType` (enum) are:
    - `resolve.EXPORT_LUT_17PTCUBE`
    - `resolve.EXPORT_LUT_33PTCUBE`
    - `resolve.EXPORT_LUT_65PTCUBE`
    - `resolve.EXPORT_LUT_PANASONICVLUT`

Unsupported Resolve API Functions
---------------------------------
The following API (functions and parameters) are no longer supported. Use job IDs instead of indices.

### Project

| Method                                      | Return Type        | Comment                                                                                                                                                                                                                                         |
|---------------------------------------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ~~`StartRendering(index1, index2, ...)`~~   | ~~`Bool`~~         | ~~Please use unique job ids (string) instead of indices.~~                                                                                                                                                                                          |
| ~~`StartRendering([idxs...])`~~             | ~~`Bool`~~         | ~~Please use unique job ids (string) instead of indices.~~                                                                                                                                                                                          |
| ~~`DeleteRenderJobByIndex(idx)`~~           | ~~`Bool`~~         | ~~Please use unique job ids (string) instead of indices.~~                                                                                                                                                                                          |
| ~~`GetRenderJobStatus(idx)`~~               | ~~`{status info}`~~| ~~Please use unique job ids (string) instead of indices.~~                                                                                                                                                                                          |
| ~~`GetSetting and SetSetting`~~             | ~~`{}`~~           | ~~settingName `videoMonitorUseRec601For422SDI` is now replaced with `videoMonitorUseMatrixOverrideFor422SDI` and `videoMonitorMatrixOverrideFor422SDI`. `perfProxyMediaOn` is now replaced with `perfProxyMediaMode`, which takes values: 0 - disabled, 1 - when available, 2 - when source not available.~~                                        |
