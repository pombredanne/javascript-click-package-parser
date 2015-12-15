# javascript-click-package-parser

A simple parser in client-side JavaScript for Ubuntu click packages.

Requires Zlib.Gunzip from https://github.com/imaya/zlib.js/blob/master/bin/gunzip.min.js to be available.

## API:

    <input type="file" id="fi">
    var click = new ClickFile(fi.files[0]);
    click.onload(function(err) {
        // get a file list of the contents of one of the contained files in a click package
        click.getFileList("control.tar.gz", function(err, filelist) {
            console.log(filelist); // [{filename: "./manifest", file_size: 123, ...}, ...]
        });
        click.getFile("data.tar.gz", "./myapp.apparmor", function(err, data) {
            console.log(data); // {"policy_groups": ["networking"], "policy_version": 1.2}
        });
    });

### ClickFile

Pass a [File](https://developer.mozilla.org/en-US/docs/Web/API/File) object and then set the `onload` method.

    var click = new ClickFile(someFileInput.files[0]);
    click.onload(function(err) {
        // read files here
    });

### ClickFile.onload()

Called with `err` parameter if load failed. `getFile` and `getFileList` will fail if called before `onload` has fired.

### ClickFile.getFileList(fileInClick, function(err, list) {...})

A click package actually contains two main files, `control.tar.gz` and `data.tar.gz`. The package manifest is in `control.tar.gz`; the app's actual code and data are in `data.tar.gz`. `ClickFile.getFileList("data.tar.gz", callback)` calls the callback with `err` and `list` parameters. `list` is a list of JS objects; an individual object looks like:

    {
        "filename":"./icon.png",
        "file_mode":"0000664",
        "file_size":8016,
        "last_modified":"12631327520",
        (many more less interesting attributes)
    }

### ClickFile.getFile(fileInClick, actualFile, function(err, data) {...})

To fetch the actual content of a file, use `ClickFile.getFile`. Pass `"data.tar.gz"` or `"control.tar.gz"` as `fileInClick`, then the actual filename you want to retrieve, and the callback is called with `err` and the file data.

The filename must be one listed in the output of `getFileList`; in particular, files in the root are likely to be named `"./something"`, not just `"something"`.

The `data` is passed as a `Uint8Array`. If you are expecting it to be readable text, then convert it to a string with `var strdata = String.fromCharCode.apply(null, data)`.

[Stuart Langridge](http://www.kryogenix.org/), December 2015.
