# Parallel Asynchronous Recursive `ls`

A parallelized recursive implementation of `ls` using `fs.readdir`

## How To Use:

### Setup

1. Follow the [Node.js Setup Guide](https://github.com/crabdude/nodejs_wiki/wiki/Setup)
1. Clone the repo:

    ```bash
    git clone git@github.com:CrabDude/nodejs-ls.git
    ```

### Development

1. Place all your code in `ls.js`'s `async function ls()`:

    ```node
    require('./helper')

    async function ls() {
      // Use 'await' inside 'async function's
      console.log('Executing ls function...')
      
      // Your implementation here
    }

    ls()
    ```
1. Run:

    ```bash
    babel-node ./ls.js
    ```
## Week 1 Challenge - Parallel Asynchronous Recursive `ls`

For this exercise, you will build a **parallelized recursive [`ls`](http://www.freebsd.org/cgi/man.cgi?query=ls&sektion=1&manpath=redhat)**, which is a CLI for listing the files in a directory.

The purpose of this exercise is to **practice control-flow** for asynchronous IO, specifically running operations in [serial](https://github.com/crabdude/nodejs_wiki/wiki/Control-flow#serial) and [parallel](https://github.com/crabdude/nodejs_wiki/wiki/Control-flow#parallel). Additionally, this exercise will explore the [`fs` filesystem](https://nodejs.org/docs/latest/api/fs.html) module from core.

**IMPORTANT: Review the [Control-flow Guide](https://github.com/crabdude/nodejs_wiki/wiki/Control-flow) to familiarize yourself with `async/await`. Ignore `Promise` and callbacks for now.**

#### Getting Started

The checkpoints below should be implemented as pairs. In pair programming, there are two roles: supervisor and driver.

The supervisor makes the decision on what step to do next. Their job is to describe the step using high level language ("Let's print out something when the user is scrolling"). They also have a browser open in case they need to do any research. The driver is typing and their role is to translate the high level task into code ("Set the scroll view delegate, implement the didScroll method).

After you finish each checkpoint, switch the supervisor and driver roles. The person on the right will be the first supervisor.

#### Milestones

0. Setup:
    - Complete the steps in the [Setting Up Nodejs](https://github.com/crabdude/nodejs_wiki/wiki/Setup) Guide, especially the [installations steps for `nodemon` and `npm-do`](https://github.com/crabdude/nodejs_wiki/wiki/Setup#install-the-nodemon-daemon-npm-do-helper-script).
    - If you haven't already, globally install `babel-node`:

        ```bash
        npm install -g babel-cli babel-preset-nodev6
        ```
    - Clone the [`ls` Starter Project](https://github.com/CrabDude/nodejs-ls).

        *Note: Your logic will go in `ls.js`, which contains the following:*
        
        ```node
        require('./helper')
        
        async function ls() {
            // Use 'await' inside 'async function's
            console.log('Executing ls function...')
            
            // Your implementation here
        }
        
        ls()
        ```
        *Note: The function name `ls()` has no special meaning, and could just as easily be named `main()` or [immediately invoked](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)*
    - Install the project's dependencies:

        ```
        project_root$ npm install
        ```
    - Run and verify your script's output:

        ```bash
        $ babel-node ls.js
        Executing ls function...
        ```
    - For convenience, automatically re-run your script on save using `nodemon`:

        ```bash
        $ nodemon --exec babel-node -- -- ls.js
        Executing ls function...
        ```
0. Implement a CLI for `fs.readdir`
    - Require (the promisified version of) the [`fs` module](https://nodejs.org/docs/latest/api/fs.html): 

        ```node
        let fs = require('fs').promise
        ```
    - To get the list of files in a directory, use [`fs.readdir`](http://nodejs.org/api/all.html#all_fs_readdir_path_callback):

        *Hint: [`__dirname`](https://nodejs.org/docs/latest/api/globals.html#globals_dirname) contains the directory path of the current file.*

        ```node
        let fs = require('fs').promise
        
        // 'await' can only be used within an 'async function'
        async function ls() {
            // fs.readdir(...) returns a Promise representing the async IO
            // Use 'await' to wait for the Promise to resolve to a real value
            let promise = fs.readdir(__dirname)
            let fileNames = await promise
            
            // TODO: Do something with fileNames
        }
        ```
        *Note: An [`async function`](https://github.com/crabdude/nodejs_wiki/wiki/Control-flow#asyncawait) like `ls` returns a `Promise` instance, which can be `awaited`ed. Obtain the promise resolution value by `await`ing as shown in the example above.*
    - [Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) through `fileNames` and output each file name to `process.stdout` using the [`.write(stringOrBuffer)`](https://nodejs.org/api/all.html#all_writable_write_chunk_encoding_callback) method 

        Your output should look something like this (remember to separate file names with a `\n` character):
        
        ```bash
        $ babel-node ls.js
        ls.js
        node_modules
        package.json
        ```
    - Exclude subdirectories from the output using [`fs.stat`](http://nodejs.org/api/all.html#all_fs_stat_path_callback) and [`path.join`](https://nodejs.org/api/all.html#all_path_join_path1_path2)
    
        *Hint: Remember to require `path`. See the require code for `fs` above.*
    
        ```node
        for (let fileName of fileNames) {
            let filePath = path.join(__dirname, file)
            // TODO: Obtain the stat promise from fs.stat(filePath)
            // TODO: Use stat.isDirectory to exclude subdirectories
            // ...
        }
        ```
    - Allow the directory path to be passed as a CLI argument:
    
        ```bash
        $ babel-node ls.js --dir=./
        ls.js
        node_modules
        package.json
        ```
        - Install the `yargs` package:

            ```bash
            $ npm install --save yargs
            ```
        - Use the value passed in on `--dir`

            ```node
            let fs = require('fs').promise
            let {dir} = require('yargs').argv // Destructuring syntax
            
            
            // Update fs.readdir() call to use dir
            async function ls(){
                // ...
                let fileNames = await fs.readdir(dir)
                // ...
            }
            // ...
            ```
            
            *Note:* See [MDN's "Destructuring assignment"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) documentation.
        - If no value for `--dir` is given, default to the current directory:

            ```node
            let {dir} = require('yargs')
                .default('dir', __dirname)
                .argv
            ```
        - Verify output of `babel-node ls.js --dir path/to/some/dir`
0. Extend the CLI to be recursive.
    - To implement recursion, the code needs to be restructured:
        - **Current** logic:
            - Call `fs.readdir(dir)`
            - Iteratively `fs.stat` the resulting `filePath`s
            - Log files
            - Ignore sub-directories
        - **Recursive** logic:
            - Pass the current directory to `ls` on the argument `rootPath`
            - `fs.stat(rootPath)`
            - If `rootPath` is a file, log and early return
            - Else, recurse on all `filePath`s returned from `fs.readdir(rootPath)`
    - Pass `dir` to `ls()`. Name the argument `rootPath`.
    
        To do this, create a separate function main and pass `dir` to 'ls' as a function parameter:
    
        ```node
        async function ls(rootPath) {
            // ...        
        }
        
        async function main() {
            // Call ls() and pass dir, remember to await
            await ls(dir)
        }
        
        // Replace ls() call with main()
        main()
        ```
    - If `rootPath` is a file, log and early return:
    
        ```node
        async function ls(rootPath) {
            // TODO: log rootPath if it's a file, then early return
            // ...        
        }
        ```
    - Recursively call `ls()` with `filePath` on subdirectories:
    
        ```node
        async function ls(rootPath) {
            // ...
            // TODO: Get 'fileNames' from fs.readdir(rootPath)
            for (let fileName of fileNames) {
                // Recurse on all files
                // Process every 'ls' call in serial (one at a time)
                // By 'await'ing on each call to 'ls'
                // This maintains output ordering
                await ls(filePath)
            }
        }
        ```
    - Ordering is nice, but performance is better. Parallelize the traversal by removing the `await` call before `ls`:
    
        ```node
        async function ls(rootPath) {
            // ...
            // TODO: Get 'fileNames' from fs.readdir(rootPath)
            for (let fileName of fileNames) {
                // Removing await recursively lists subdirectories in parallel
                ls(filePath)
            }
        }
        ```
    - Verify your output
0. *Bonus:* Return a flat array of file paths instead of printing them as you go:
    - Return an array of file paths for both single files and directories:

        ```node
        // Single file case (return instead of logging)
        return [rootPath]
        
        // Sub-directory case
        let lsPromises = []
        for (let fileName of fileNames) {
            // ...
            let promise = ls(filePath)
            lsPromises.push(promise)
        }
        // The resulting array needs to be flattened
        return await Promise.all(lsPromises)
        ```
        
        *Note:* To `await` several asynchronous operations (`Promise`s) in parallel (as opposed to in serial, aka one at a time), use [`Promise.all`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) like so: `await Promise.all(arrayOfPromises)`.
    - Concatenate the results with [`Array.prototype.concat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat) or use [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce#Flatten_an_array_of_arrays) to flatten the resulting recursive arrays.
    - Print the results (return value of `ls(dir)`) with a single `console.log`:

        ```node
        async function main() {
            let filePaths = await ls(rootPath)
            // TODO: Output filePaths
        }
        ```
0. *Bonus:* Execute `ls.js` directly.
    To make a node.js / JavaScript file executable:

    1. Mark the file as [executable](https://en.wikipedia.org/wiki/File_system_permissions#Permissions):
    
        ```bash
        $ chmod +x ./ls.js
        ```
    2. Add a [node.js](http://dailyjs.com/2012/03/01/unix-node-arguments/) [shebang](http://dailyjs.com/2012/03/01/unix-node-arguments/) by appending the following to the top of `ls.js`:

        **Linux / OSX:**
        
        ```node
        #!/usr/bin/env babel-node
        ```
        
        [**Windows** is a little more complicated...](http://whitescreen.nicolaas.net/programming/windows-shebangs)
    3. Verify by running `ls.js` without `node`:

        ```bash
        $ ./ls.js --dir=./
        ls.js
        node_modules
        package.json
        ```

    
    

### Guides

- [Filesystem API](https://github.com/crabdude/nodejs_wiki/wiki/Filesystem)
- [Control-flow](https://github.com/crabdude/nodejs_wiki/wiki/Control-flow)
- [CLI](https://github.com/crabdude/nodejs_wiki/wiki/CLI)
- [Setup](https://github.com/crabdude/nodejs_wiki/wiki/Setup)

### References

- [`fs`](https://nodejs.org/docs/latest/api/fs.html)
- [`nodemon`](http://npmjs.com/package/nodemon)
- [`babel-cli`](http://npmjs.com/package/babel-cli)
- [`babel-preset-nodev6`](http://npmjs.com/package/babel-preset-nodev6)

