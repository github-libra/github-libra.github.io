---
title: Automation by grunt
category: process
---

## Install

Install `grunt` and `grunt-cli`. `grunt-cli` runs the version of Grunt which has been installed next to a Gruntfile. This allows multiple versions of Grunt to be installed on the same machine simultaneously.

## The Gruntfile.js

It can be Gruntfile.js or Gruntfile.coffee. It compromises of following parts:

* The "wrapper" function
* Project and task configuration
* Loading Grunt plugins and tasks
* Custom tasks

```javascript
module.exports = function(grunt) {// The "wrapper" function.

  // Project configuration.
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    uglify: {
      options: {
        banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
      },
      build: {
        src: 'src/<%= pkg.name %>.js',
        dest: 'build/<%= pkg.name %>.min.js'
      }
    }
  });

  // Load the plugin that provides the "uglify" task.
  grunt.loadNpmTasks('grunt-contrib-uglify');

  // custom task.
  grunt.registerTask('default', ['uglify']);

};

```

## Configure tasks

Configure tasks by `grunt.initConfig()`, usually it's the task name, it will ignore any custom data when configuring.

```javascript
grunt.initConfig({
  concat: {
    // concat task configuration goes here.
  },
  uglify: {
    // uglify task configuration goes here.
  },
  // Arbitrary non-task-specific properties.
  my_property: 'whatever',
  my_src_files: ['foo/*.js', 'bar/*.js'],
});
```

Every task can have serveral `targets` with different config. `grunt concat:dev` runs against dev target. `grunt concat` runs against all the targets in turn.

>Inside a task configuration, an `options` property may be specified to override built-in defaults. In addition, each target may have an `options` property which is specific to that target. Target-level options will override task-level options.
The `options` object is optional and may be omitted if not needed.

### Files to process

Apart from `src` and `dest`, you can also specify multi files by `files object` where the key is destination and value is src files.

Use `files array` format if you want to add additional property for each mapping.

```javascript
grunt.initConfig({
  concat: {
    foo: {
      files: [
        {src: ['src/aa.js', 'src/aaa.js'], dest: 'dest/a.js'},
        {src: ['src/aa1.js', 'src/aaa1.js'], dest: 'dest/a1.js'},
      ],
    },
    bar: {
      files: [
        {src: ['src/bb.js', 'src/bbb.js'], dest: 'dest/b/', nonull: true},
        {src: ['src/bb1.js', 'src/bbb1.js'], dest: 'dest/b1/', filter: 'isFile'},
      ],
    },
  },
});
```

The `filter` can also be overrided by return true/false.

### globbing pattern

It allows using glob to give files:

* `\*` matches any number of characters, but not `/`
* `?` matches a single character, but not `/`
* `**` matches any number of characters, including `/`, as long as it's the only thing in a path part
* `{}` allows for a comma-separated list of "or" expressions
* `!` at the beginning of a pattern will negate the match

### Build files dynamically

Set `expand` to true allows use of following properties:

* `cwd` All src matches are relative to (but don't include) this path.
* `src` Pattern(s) to match, relative to the cwd.
* `dest` Destination path prefix.
* `ext` Replace any existing extension with this value in generated dest paths.
* `extDot` Used to indicate where the period indicating the extension is located. Can take either 'first' (extension begins after the first period in the file name) or 'last' (extension begins after the last period), and is set by default to 'first' [Added in 0.4.3]
* `flatten` Remove all path parts from generated dest paths.
* `rename` Embeds a customized function, which returns a string containing the new destination and filename. This function is called for each matched src file (after extension renaming and flattening).

```javascript
target1: {
  // Grunt will search for "**/*.js" under "lib/" when the "uglify" task
  // runs and build the appropriate src-dest file mappings then, so you
  // don't need to update the Gruntfile when files are added or removed.
  files: [
    {
      expand: true,     // Enable dynamic expansion.
      cwd: 'lib/',      // Src matches are relative to this path.
      src: ['**/*.js'], // Actual pattern(s) to match.
      dest: 'build/',   // Destination path prefix.
      ext: '.min.js',   // Dest filepaths will have this extension.
      extDot: 'first'   // Extensions in filenames begin after the first dot
    },
  ],
}
```

Templates can also be used in configuration. Expressions inside `<%= %>` will be eveluate against the entire config object. Templates are expanded recursively until no more remain.

Use `grunt.file.readJSON` and `grunt.file.readYAML` to load external data.


## Register task

`grunt.registerTask(taskName, [description, ] taskList)` to declare a task. Task can call other tasks inside it.

When tasks fail, all subsequent tasks will be aborted unless --force was specified.

```javascript
grunt.registerTask('foo', 'My "foo" task.', function() {
  // Fail synchronously.
  return false;
});

grunt.registerTask('bar', 'My "bar" task.', function() {
  var done = this.async();
  setTimeout(function() {
    // Fail asynchronously.
    done(false);
  }, 1000);
});
```

`grunt.task.requires()` can be used to make sure some tasks has ran successfully.
`grunt.config.requires()` can be used to check come config is right.

```javascript
grunt.registerTask('foo', 'My "foo" task.', function() {
  grunt.task.requires('foo');
  // Fail task if "meta.name" config prop is missing
  // Format 1: String 
  grunt.config.requires('meta.name');
  // or Format 2: Array
  grunt.config.requires(['meta', 'name']);
  // Log... conditionally.
  grunt.log.writeln('This will only log if meta.name is defined in the config.');
  // access config
  grunt.log.writeln('The meta.name property is: ' + grunt.config('meta.name'));
  // access env config
  grunt.log.writeln(process.env);

});
```












