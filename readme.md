# Fileable

Render a file tree using a JSX template.

Inspired by [React FS Renderer](https://github.com/ericvicenti/react-fs-renderer)

## API -- Template

Fileable can render [functional components](https://reactjs.org/docs/components-and-props.html) built with the following components:

### Component: File

The File component represents a file.

```javascript
import {File} from 'fileable';
const template = ()=><File name='readme.md'>
# This is a sample file.
</File>
```

A file may have contain other files. Content will be concatinated.

```javascript
const template = ()=><File name='readme.md'>
    # This is a sample file.
    <File>## This is more content</File>
    <File>## As is this</File>
    ## This is the end
</File>
```

#### File Attribute: name

If a file is not the child of another file, this will be used as it's name.

```javascript
const template = ()=><File name="empty_file"/>;
```

If no name is passed, a hash of the content will be used.

By default, the algorithm used is SHA_256, but a number of algorithms are available.

```javascript
import {FILE_NAME_MD5, FILE_NAME_SHA_1, FILE_NAME_SHA_256, FILE_NAME_SHA_512, FILE_NAME_SHA_3} from 'fileable';
const template = ()=><File name={FILE_NAME_MD5}/>;
```

#### File Attribute: extension

Extension appended to name.
Particularly useful when name is autogenerated.

```javascript
const template = ()=><File extension='.js'>
...javascript content
</File>;
```

#### File Attribute: src

File content in src attribute will be appended to child content.

```javascript
const template = ()=><File src='https://www.google.com'>Google's Home Page:</File>;
```

If src location is a local file, it is treated as so relative to given template.

```javascript
const template = ()=><File src='./sibbling-of-template'/>;
```

#### File Attribute: cmd

Command will be run and content in src attribute will be appended to child content.

```javascript
const template = ()=><File cmd='date'>Content created at:</File>;
```

If cmd starts with '|', child content will be piped through command

```javascript
const template = ()=><File name='.cheesefile' cmd='|grep Cheese'>
Lines containing "Cheese:":
    - Feta Cheese
    - Broccoli
    - American Cheese
    - Ham
    - Bread
    - Cheesebread
</File>;
```

#### File Attribute: mode

Set the mode of a file.

```javascript
const template = ()=><File name='helloworld' mode={0o777}>
#!/usr/bin/env node
console.log('hello world');
</File>;
```

#### File Attribute: doctype

Add a doctype preamble the mode of a file.

```javascript
const template = ()=><File name='index.html' doctype='html'>
{`<html>
    <head></head>
    <body></body>
</html>`}
</File>;
```

#### File Attribute: transform

Command will be transformed via given function

```javascript

const addBeginning = (content)=>
`// Begin$
${content}`;

const template = ()=><File transform={addBeginning}>
// Middle
</File>;
```

#### File Attribute: append

When used within other files, the append adds content to files after any transfomations from (cmd, transform) are applied.

```javascript

const addEnd = (content)=>
`${content}
// ^End`;

const template = ()=><File transform={addBeginning}>
    // Middle
    <File append/>
    // Post End
    </File>
</File>;
```

#### File Attribute: clone

When used within other files, the clone command takes the content of the file up to that point and clones it to another file.

```javascript
const template = ()=><File name='main.js'>
    Content
    <File clone />
</File>;
```

If a placeholder string is passed as the clone attribute, it will appear in place of that file within the parent.

```javascript
const template = ()=><File name='main.js'>
    Content
    <File clone='// filed cloned at hash' />
</File>;
```

Combine other attributes to create "sidecar" files

```javascript
const template = ()=><File src='../src/index.js' name='index.js' cmd='|./minify-file'>
    <File clone='//src-map: index.map.js' append name='index.map.js' cmd='|./produce-file-map'></File>
</File>;
```

#### HTML Content

[React's HTML tags](https://react-cn.github.io/react/docs/tags-and-attributes.html) can be used to produce a file's content.

```javascript
const template = ()=> <File name="index.html">
    &lt;!doctype html&gt;
    <html>
        <Head />
        <body> Hello World
            <script src='index.js'></script>
        </body>
    </html>
</File>;
```

### Component:Folder

The Folder component represents a Folder;

```javascript
import {Folder} from 'fileable';
const template = ()=><Folder name='project/'/>
```

Folders can contain files and other folders

```javascript
const template = ()=><Folder name='project/'>
    <File name='empty_file'/>
    <Folder name='empty_folder'/>
</Folder>
```

#### Folder Attribute: name

Similar to File name.

A folder must have a name attribute.
In the future, we may recursively create a folder's name based on the hash of a folder's conetent.

#### Folder Attribute: zip

Archives can be created with using the zip attribute

```javascript
const template = ()=><Folder name='project' zip>
    <File name='empty_file'/>
</Folder>
```

#### Folder Attribute: extension

Similar to File extension.

```javascript
const template = ()=><Folder name='project' zip extension='.zip'>
    <File name='empty_file' />
</Folder>;
```

### Component:Clear

Wrapping components within a Clear component removes __every single file__ from the given folder context before writing.

```javascript
import {Clear} from 'fileable';
const template = ()=><Clear><File name='empty_file' /></Clear>;
```

#### Attribute: protect

The protect attribute can be used to specifiy files not to be removed.
It can take the form of a specific file name, a file glob, or a regular expression.

```javascript
import {Clear} from 'fileable';
const template = ()=><Clear protect='**/*.md'><File name='empty_file' /></Clear>;
```

#### Attribute: only

Like protect, but files specified in the protect attribute are removed while others are ignored.

```javascript
import {Clear} from 'fileable';
const template = ()=><Clear only={new RegExp(/\.js$/)}><File name='empty_file' /></Clear>;
```

### Component: React.Fragment

React.Fragment can be used to group files and folders

```javascript
import {Fragment} from 'react';
const template = ()=><>
    <File name='empty_file'/>
    <Folder name='empty_folder'/>
</>;
```

## API -- CLI

Fileable's renderer's can be used from the command line

### CLI Installation

```sh
npm install --global fileable
```

### CLI Usage

```sh
fileable build <template> <destination>
```

```sh
fileable build ./template.jsx ./dist
```

#### --test/--no-test flag


```sh
fileable build ./template.jsx ./dist --no-test
```

#### --input flag

```sh
fileable build ./template.jsx ./dist --no-test
```

#### --help

Try `fileable --help` for more options

#### Remote Files

Files that begin with 'http://', 'https://', or 'ftp://' can be used as input or template files with a few creavats.

For remote files, DO NOT import of fileable components (File, Folder, Clear), but you'll still have to import compents used for react.  In the future, you'll likey import components from another library, but for now, please bear with us.

```javascript
import React, {Fragment} from 'react';
export default ()=><File>...
```
## API

### Table of contents

- [function Clear](#function-clear)
  - [Examples](#examples)
- [function File](#function-file)
  - [Examples](#examples-1)
- [function Folder](#function-folder)
  - [Examples](#examples-2)
- [function iterator](#function-iterator)
  - [Examples](#examples-3)
- [function renderConsole](#function-renderconsole)
  - [Examples](#examples-4)
- [function renderFS](#function-renderfs)
  - [Examples](#examples-5)

### function Clear

Clear Component

#### Examples

> ```javascript
>  // template.jsx
>  import {File, Folder, Clear} from 'fileable';
>  export () => <Clear><Folder name='project'><File name='readme.md'/></Folder></Clear>
> ```

* * *

### function File

File component

#### Examples

> ```javascript
> // template.jsx
>  import {File} from 'fileable';
>  export () => <File name='readme.md'/>
> ```

* * *

### function Folder

Folder component

#### Examples

> ```javascript
>  // template.jsx
>  import {File, Folder} from 'fileable';
>  export () => <Folder name='project'><File name='readme.md'/></Folder>
> ```

* * *

### function iterator

Iterator

| Parameter | Type   | Description |
| :-------- | :----- | :---------- |
| `input`   | object |             |

#### Examples

> ```javascript
> import {iterator} from 'fileable';
> ```

* * *

### function renderConsole

Render to Console

| Parameter | Type   | Description |
| :-------- | :----- | :---------- |
| `input`   | object |             |

#### Examples

> ```javascript
> import {renderConsole} from 'fileable';
> const main = async () =>
> renderConsole(template(), { folder_context: [directory] });
> main();
> ```

* * *

### function renderFS

Render to File System

| Parameter | Type   | Description |
| :-------- | :----- | :---------- |
| `input`   | object |             |

#### Examples

> ```javascript
> import {renderFS} from 'fileable';
> const main = async () =>
> renderFS(template(), { folder_context: [directory] });
> main();
> ```

## Todo

- Asynchronous content
    - Useful for extending
- Make work like described API in readme.
- Handle newline trickiness
    - inability to insert new lines easily
    - must manually add "{'\n'}" or enclose entirely witin backticks ("'``'")
- Eventually, get remote files working with using dynamic imports
