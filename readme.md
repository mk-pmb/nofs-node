# [nofs](https://github.com/ysmood/nofs)

## Overview

`nofs` extends Node's native `fs` module with some useful methods. It tries
to make your functional programming experience better.

[![NPM version](https://badge.fury.io/js/nofs.svg)](http://badge.fury.io/js/nofs) [![Build Status](https://travis-ci.org/ysmood/nofs.svg)](https://travis-ci.org/ysmood/nofs) [![Build status](https://ci.appveyor.com/api/projects/status/11ddy1j4wofdhal7?svg=true)](https://ci.appveyor.com/project/ysmood/nofs)
 [![Deps Up to Date](https://david-dm.org/ysmood/nofs.svg?style=flat)](https://david-dm.org/ysmood/nofs)

## Features

- Introduce `map` and `reduce` to folders.
- Recursive `glob`, `move`, `copy`, `remove`, etc.
- **Promise** by default.
- Unified API. Support **Promise**, **Sync** and **Callback** paradigms.

## Install

```shell
npm install nofs
```

## API Convention

### Unix Path Separator

When the system is Windows and `process.env.force_unix_sep != 'off'`, nofs  will force all the path separator to `/`, such as `C:\a\b` will be transformed to `C:/a/b`.

### Promise, Sync and Callback

Any function that has a `Sync` version will has a promise version that ends with `P`.
For example the `fs.remove` will have `fs.removeSync` for sync IO, and `fs.removeP` for Promise.

### `eachDir`

It is the core function for directory manipulation. Other abstract functions
like `mapDir`, `reduceDir`, `readDirs` are built on top of it. You can play
with it if you don't like other functions.

### `nofs` vs Node Native `fs`

`nofs` only extends the native module, no pollution will be found. You can
still call `nofs.readFile` as easy as pie.

### Inheritance of Options

A Function's options may inherit other function's, especially the functions it calls internally. Such as the `readDirs` extends the `eachDir`'s
option, therefore `readDirs` also has a `filter` option.

## Quick Start

```coffee
# You can replace "require('fs')" with "require('nofs')"
fs = require 'nofs'

# Callback
fs.outputFile 'x.txt', 'test', (err) ->
    console.log 'done'

# Sync
fs.readFileSync 'x.txt'
fs.copySync 'dir/a', 'dir/b'

# Promise
fs.mkdirsP 'deep/dir/path'
.then ->
    fs.outputFileP 'a.txt', 'hello world'
.then ->
    fs.moveP 'a.txt', 'b.txt'
.then ->
    fs.copyP 'b.txt', 'c.js'
.then ->
    # Get all txt files.
    fs.globP 'deep/**'
.then (list) ->
    console.log list
.then ->
    fs.removeP 'deep', { filter: '**/*.js' }
```

## Changelog

Goto [changelog](doc/changelog.md)

## API

__No native `fs` funtion will be listed.__

### nofs

- #### <a href="src/main.coffee?source#L5" target="_blank"><b>Overview</b></a>

  I hate to reinvent the wheel. But to purely use promise, I don't
  have many choices.

- #### <a href="src/main.coffee?source#L16" target="_blank"><b>Promise</b></a>

  Here I use [Bluebird][Bluebird] only as an ES6 shim for Promise.
  No APIs other than ES6 spec will be used. In the
  future it will be removed.
  [Bluebird]: https://github.com/petkaantonov/bluebird

- #### <a href="src/main.coffee?source#L51" target="_blank"><b>copyDirP</b></a>

  Copy an empty directory.

  - **<u>param</u>**: `src` { _String_ }

  - **<u>param</u>**: `dest` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    ```coffee
    {
    	isForce: false
    	mode: auto
    }
    ```

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L76" target="_blank"><b>copyDirSync</b></a>

  See `copyDirP`.

- #### <a href="src/main.coffee?source#L113" target="_blank"><b>copyFileP</b></a>

  Copy a single file.

  - **<u>param</u>**: `src` { _String_ }

  - **<u>param</u>**: `dest` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    ```coffee
    {
    	isForce: false
    	mode: auto
    }
    ```

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L150" target="_blank"><b>copyFileSync</b></a>

  See `copyDirP`.

- #### <a href="src/main.coffee?source#L203" target="_blank"><b>copyP</b></a>

  Like `cp -r`.

  - **<u>param</u>**: `from` { _String_ }

    Source path.

  - **<u>param</u>**: `to` { _String_ }

    Destination path.

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`.
    Defaults:
    ```coffee
    {
    	# Overwrite file if exists.
    	isForce: false
    }
    ```

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L237" target="_blank"><b>copySync</b></a>

  See `copyP`.

- #### <a href="src/main.coffee?source#L271" target="_blank"><b>dirExistsP</b></a>

  Check if a path exists, and if it is a directory.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>return</u>**:  { _Promise_ }

    Resolves a boolean value.

- #### <a href="src/main.coffee?source#L281" target="_blank"><b>dirExistsSync</b></a>

  Check if a path exists, and if it is a directory.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>return</u>**:  { _boolean_ }

- #### <a href="src/main.coffee?source#L382" target="_blank"><b>eachDirP</b></a>

  Walk through a path recursively with a callback. The callback
  can return a Promise to continue the sequence. The resolving order
  is also recursive, a directory path resolves after all its children
  are resolved.

  - **<u>param</u>**: `spath` { _String_ }

    The path may point to a directory or a file.

  - **<u>param</u>**: `opts` { _Object_ }

    Optional. Defaults:
    ```coffee
    {
    	# Include entries whose names begin with a dot (.).
    	all: true
    
    	# To filter paths. It can also be a RegExp or a glob pattern string.
    	# When it's a string, it extends the Minimatch's options.
    	filter: (fileInfo) -> true
    
    	# The current working directory to search.
    	cwd: ''
    
    	# Whether to include the root directory or not.
    	isIncludeRoot: true
    
    	# Whehter to follow symbol links or not.
    	isFollowLink: true
    
    	# Iterate children first, then parent folder.
    	isReverse: false
    
    	# When isReverse is false, it will be the previous fn resolve value.
    	val: any
    
    	# If it return false, sub-entries won't be searched.
    	# When the `filter` option returns false, its children will
    	# still be itered. But when `searchFilter` returns false, children
    	# won't be itered by the fn.
    	searchFilter: (fileInfo) -> true
    
    	# Such as force `C:\test\path` to `C:/test/path`.
    	# This option only works on Windows.
    	isForceUnixSep: isWin and process.env.force_unix_sep == 'off'
    }
    ```

  - **<u>param</u>**: `fn` { _Function_ }

    `(fileInfo) -> Promise | Any`.
    The `fileInfo` object has these properties: `{ path, isDir, children, stats }`.
    Assume the `fn` is `(f) -> f`, the directory object array may look like:
    ```coffee
    {
    	path: 'dir/path'
    	name: 'path'
    	isDir: true
    	val: 'test'
    	children: [
    		{ path: 'dir/path/a.txt', name: 'a.txt', isDir: false, stats: { ... } }
    		{ path: 'dir/path/b.txt', name: 'b.txt', isDir: false, stats: { ... } }
    	]
    	stats: {
    		size: 527
    		atime: Mon, 10 Oct 2011 23:24:11 GMT
    		mtime: Mon, 10 Oct 2011 23:24:11 GMT
    		ctime: Mon, 10 Oct 2011 23:24:11 GMT
    		...
    	}
    }
    ```
    The `stats` is a native `nofs.Stats` object.

  - **<u>return</u>**:  { _Promise_ }

    Resolves a directory tree object.

  - **<u>example</u>**:

    ```coffee
    # Print all file and directory names, and the modification time.
    nofs.eachDirP 'dir/path', (obj, stats) ->
    	console.log obj.path, stats.mtime
    
    # Print path name list.
    nofs.eachDirP 'dir/path', (curr) -> curr
    .then (tree) ->
    	console.log tree
    
    # Find all js files.
    nofs.eachDirP 'dir/path', {
    	filter: '**/*.js', nocase: true
    }, ({ path }) ->
    	console.log paths
    
    # Find all js files.
    nofs.eachDirP 'dir/path', { filter: /\.js$/ }, ({ path }) ->
    	console.log paths
    
    # Custom filter
    nofs.eachDirP 'dir/path', {
    	filter: ({ path, stats }) ->
    		path.slice(-1) != '/' and stats.size > 1000
    }, (path) ->
    	console.log path
    ```

- #### <a href="src/main.coffee?source#L461" target="_blank"><b>eachDirSync</b></a>

  See `eachDirP`.

  - **<u>return</u>**:  { _Object | Array_ }

    A tree data structure that
    represents the files recursively.

- #### <a href="src/main.coffee?source#L545" target="_blank"><b>fileExistsP</b></a>

  Check if a path exists, and if it is a file.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>return</u>**:  { _Promise_ }

    Resolves a boolean value.

- #### <a href="src/main.coffee?source#L555" target="_blank"><b>fileExistsSync</b></a>

  Check if a path exists, and if it is a file.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>return</u>**:  { _boolean_ }

- #### <a href="src/main.coffee?source#L595" target="_blank"><b>globP</b></a>

  Get files by patterns.

  - **<u>param</u>**: `pattern` { _String | Array_ }

    The minimatch pattern.

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`.
    But the `filter` property will be fixed with the pattern.
    Defaults:
    ```coffee
    {
    	all: false
    
    	# The minimatch option object.
    	minimatch: {}
    }
    ```

  - **<u>param</u>**: `fn` { _Function_ }

    `(fileInfo, list) -> Promise | Any`.
    It will be called after each match. By default it is:
    `(fileInfo, list) -> list.push fileInfo.path`

  - **<u>return</u>**:  { _Promise_ }

    Resolves the list array.

  - **<u>example</u>**:

    ```coffee
    # Get all js files.
    nofs.globP('**/*.js').then (paths) ->
    	console.log paths
    
    # Custom the iterator. Append '/' to each directory path.
    nofs.globP('**/*.js', (info, list) ->
    	list.push if info.isDir
    		info.path + '/'
    	else
    		info.path
    ).then (paths) ->
    	console.log paths
    ```

- #### <a href="src/main.coffee?source#L655" target="_blank"><b>globSync</b></a>

  See `globP`.

  - **<u>return</u>**:  { _Array_ }

    The list array.

- #### <a href="src/main.coffee?source#L735" target="_blank"><b>mapDirP</b></a>

  Map file from a directory to another recursively with a
  callback.

  - **<u>param</u>**: `from` { _String_ }

    The root directory to start with.

  - **<u>param</u>**: `to` { _String_ }

    This directory can be a non-exists path.

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`. But `cwd` is
    fixed with the same as the `from` parameter.

  - **<u>param</u>**: `fn` { _Function_ }

    `(src, dest, fileInfo) -> Promise | Any` The callback
    will be called with each path. The callback can return a `Promise` to
    keep the async sequence go on.

  - **<u>return</u>**:  { _Promise_ }

    Resolves a tree object.

  - **<u>example</u>**:

    ```coffee
    # Copy and add license header for each files
    # from a folder to another.
    nofs.mapDirP(
    	'from'
    	'to'
    	(src, dest, fileInfo) ->
    		return if fileInfo.isDir
    		nofs.readFileP(src).then (buf) ->
    			buf += 'License MIT\n' + buf
    			nofs.writeFileP dest, buf
    )
    ```

- #### <a href="src/main.coffee?source#L751" target="_blank"><b>mapDirSync</b></a>

  See `mapDirP`.

  - **<u>return</u>**:  { _Object | Array_ }

    A tree object.

- #### <a href="src/main.coffee?source#L769" target="_blank"><b>minimatch</b></a>

  The `minimatch` lib.
  [Documentation](https://github.com/isaacs/minimatch)
  [Offline Documentation](?gotoDoc=minimatch/readme.md)

  - **<u>type</u>**:  { _Funtion_ }

- #### <a href="src/main.coffee?source#L777" target="_blank"><b>mkdirsP</b></a>

  Recursively create directory path, like `mkdir -p`.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `mode` { _String_ }

    Defaults: `0o777 & ~process.umask()`

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L800" target="_blank"><b>mkdirsSync</b></a>

  See `mkdirsP`.

- #### <a href="src/main.coffee?source#L823" target="_blank"><b>moveP</b></a>

  Moves a file or directory. Also works between partitions.
  Behaves like the Unix `mv`.

  - **<u>param</u>**: `from` { _String_ }

    Source path.

  - **<u>param</u>**: `to` { _String_ }

    Destination path.

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`.
    Defaults:
    ```coffee
    {
    	isForce: false
    }
    ```

  - **<u>return</u>**:  { _Promise_ }

    It will resolve a boolean value which indicates
    whether this action is taken between two partitions.

- #### <a href="src/main.coffee?source#L857" target="_blank"><b>moveSync</b></a>

  See `moveP`.

- #### <a href="src/main.coffee?source#L895" target="_blank"><b>outputFileP</b></a>

  Almost the same as `writeFile`, except that if its parent
  directories do not exist, they will be created.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `data` { _String | Buffer_ }

  - **<u>param</u>**: `opts` { _String | Object_ }

    Same with the `nofs.writeFile`.

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L907" target="_blank"><b>outputFileSync</b></a>

  See `outputFileP`.

- #### <a href="src/main.coffee?source#L930" target="_blank"><b>outputJsonP</b></a>

  Write a object to a file, if its parent directory doesn't
  exists, it will be created.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `obj` { _Any_ }

    The data object to save.

  - **<u>param</u>**: `opts` { _Object | String_ }

    Extends the options of `outputFileP`.
    Defaults:
    ```coffee
    {
    	replacer: null
    	space: null
    }
    ```

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L944" target="_blank"><b>outputJsonSync</b></a>

  See `outputJSONP`.

- #### <a href="src/main.coffee?source#L996" target="_blank"><b>readDirsP</b></a>

  Read directory recursively.

  - **<u>param</u>**: `root` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`. Defaults:
    ```coffee
    {
    	# Don't include the root directory.
    	isIncludeRoot: false
    
    	isCacheStats: false
    
    	all: false
    }
    ```
    If `isCacheStats` is set true, the returned list array
    will have an extra property `statsCache`, it is something like:
    ```coffee
    {
    	'path/to/entity': {
    		dev: 16777220
    		mode: 33188
    		...
    	}
    }
    ```
    The key is the entity path, the value is the `nofs.Stats` object.

  - **<u>return</u>**:  { _Promise_ }

    Resolves an path array.

  - **<u>example</u>**:

    ```coffee
    # Basic
    nofs.readDirsP 'dir/path'
    .then (paths) ->
    	console.log paths # output => ['dir/path/a', 'dir/path/b/c']
    
    # Same with the above, but cwd is changed.
    nofs.readDirsP 'path', { cwd: 'dir' }
    .then (paths) ->
    	console.log paths # output => ['path/a', 'path/b/c']
    
    # CacheStats
    nofs.readDirsP 'dir/path', { isCacheStats: true }
    .then (paths) ->
    	console.log paths.statsCache['path/a']
    ```

- #### <a href="src/main.coffee?source#L1024" target="_blank"><b>readDirsSync</b></a>

  See `readDirsP`.

  - **<u>return</u>**:  { _Array_ }

    Path string array.

- #### <a href="src/main.coffee?source#L1059" target="_blank"><b>readJsonP</b></a>

  Read A Json file and parse it to a object.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `opts` { _Object | String_ }

    Same with the native `nofs.readFile`.

  - **<u>return</u>**:  { _Promise_ }

    Resolves a parsed object.

  - **<u>example</u>**:

    ```coffee
    nofs.readJsonP('a.json').then (obj) ->
    	console.log obj.name, obj.age
    ```

- #### <a href="src/main.coffee?source#L1070" target="_blank"><b>readJsonSync</b></a>

  See `readJSONP`.

  - **<u>return</u>**:  { _Any_ }

    The parsed object.

- #### <a href="src/main.coffee?source#L1098" target="_blank"><b>reduceDirP</b></a>

  Walk through directory recursively with a callback.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`,
    with some extra options:
    ```coffee
    {
    	# The init value of the walk.
    	init: undefined
    }
    ```

  - **<u>param</u>**: `fn` { _Function_ }

    `(prev, path, isDir, stats) -> Promise`

  - **<u>return</u>**:  { _Promise_ }

    Final resolved value.

  - **<u>example</u>**:

    ```coffee
    # Concat all files.
    nofs.reduceDirP 'dir/path', { init: '' }, (val, info) ->
    	return val if info.isDir
    	nofs.readFileP(info.path).then (str) ->
    		val += str + '\n'
    .then (ret) ->
    	console.log ret
    ```

- #### <a href="src/main.coffee?source#L1117" target="_blank"><b>reduceDirSync</b></a>

  See `reduceDirP`

  - **<u>return</u>**:  { _Any_ }

    Final value.

- #### <a href="src/main.coffee?source#L1136" target="_blank"><b>removeP</b></a>

  Remove a file or directory peacefully, same with the `rm -rf`.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    Extends the options of `eachDir`. But
    the `isReverse` is fixed with `true`.

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L1151" target="_blank"><b>removeSync</b></a>

  See `removeP`.

- #### <a href="src/main.coffee?source#L1178" target="_blank"><b>touchP</b></a>

  Change file access and modification times.
  If the file does not exist, it is created.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    Default:
    ```coffee
    {
    	atime: Date.now()
    	mtime: Date.now()
    	mode: undefined
    }
    ```

  - **<u>return</u>**:  { _Promise_ }

    If new file created, resolves true.

- #### <a href="src/main.coffee?source#L1197" target="_blank"><b>touchSync</b></a>

  See `touchP`.

  - **<u>return</u>**:  { _Boolean_ }

    Whether a new file is created or not.

- #### <a href="src/main.coffee?source#L1236" target="_blank"><b>watchFileP</b></a>

  Watch a file. If the file changes, the handler will be invoked.
  You can change the polling interval by using `process.env.pollingWatch`.
  Use `process.env.watchPersistent = 'off'` to disable the persistent.
  Why not use `nofs.watch`? Because `nofs.watch` is unstable on some file
  systems, such as Samba or OSX.

  - **<u>param</u>**: `path` { _String_ }

    The file path

  - **<u>param</u>**: `handler` { _Function_ }

    Event listener.
    The handler has these params:
    - file path
    - current `nofs.Stats`
    - previous `nofs.Stats`
    - if its a deletion

  - **<u>param</u>**: `autoUnwatch` { _Boolean_ }

    Auto unwatch the file while file deletion.
    Default is true.

  - **<u>return</u>**:  { _Promise_ }

    It resolves the wrapped watch listener.

  - **<u>example</u>**:

    ```coffee
    process.env.watchPersistent = 'off'
    nofs.watchFileP 'a.js', (path, curr, prev, isDeletion) ->
    	if curr.mtime != prev.mtime
    		console.log path
    ```

- #### <a href="src/main.coffee?source#L1267" target="_blank"><b>watchFilesP</b></a>

  Watch files, when file changes, the handler will be invoked.
  It is build on the top of `nofs.watchFileP`.

  - **<u>param</u>**: `patterns` { _Array_ }

    String array with minimatch syntax.
    Such as `['*/**.css', 'lib/**/*.js']`.

  - **<u>param</u>**: `handler` { _Function_ }

  - **<u>return</u>**:  { _Promise_ }

    It contains the wrapped watch listeners.

  - **<u>example</u>**:

    ```coffee
    nofs.watchFiles '*.js', (path, curr, prev, isDeletion) ->
    	console.log path
    ```

- #### <a href="src/main.coffee?source#L1308" target="_blank"><b>watchDirP</b></a>

  Watch directory and all the files in it.
  It supports three types of change: create, modify, move, delete.
  By default, `move` event is disabled.
  It is build on the top of `nofs.watchFileP`.

  - **<u>param</u>**: `root` { _String_ }

  - **<u>param</u>**: `opts` { _Object_ }

    Defaults:
    ```coffee
    {
    	pattern: '**' # minimatch, string or array
    
    	# Whether to watch POSIX hidden file.
    	all: false
    
    	# The minimatch options.
    	minimatch: {}
    
    	isEnableMoveEvent: false
    }
    ```

  - **<u>param</u>**: `fn` { _Function_ }

    `(type, path, oldPath) ->`.
    If the "path" ends with '/' it's a directory, else a file.

  - **<u>return</u>**:  { _Promise_ }

    Resolves a object that keys are paths,
    values are listeners.

  - **<u>example</u>**:

    ```coffee
    # Only current folder, and only watch js and css file.
    nofs.watchDir {
    	dir: 'lib'
    	pattern: '*.+(js|css)'
    	handler: (type, path) ->
    		console.log type
    		console.log path
    }
    ```

- #### <a href="src/main.coffee?source#L1393" target="_blank"><b>writeFileP</b></a>

  A `writeFile` shim for `< Node v0.10`.

  - **<u>param</u>**: `path` { _String_ }

  - **<u>param</u>**: `data` { _String | Buffer_ }

  - **<u>param</u>**: `opts` { _String | Object_ }

  - **<u>return</u>**:  { _Promise_ }

- #### <a href="src/main.coffee?source#L1418" target="_blank"><b>writeFileSync</b></a>

  See `writeFileP`



## Function Alias

For some naming convention reasons, `nofs` also uses some common alias for fucntions. See [src/alias.coffee](src/alias.coffee).

## Benckmark

[`nofs.copy` vs `ncp`](benchmark/ncp.coffee)

## Lisence

MIT