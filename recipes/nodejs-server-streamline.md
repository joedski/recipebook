node.js Server using Streamline.js
==================================

> node.js version as of writing: `5.4.1`.  This recipe uses Gulp as the build system/taskrunner.

Required Deps:
- `streamline-runtime`

Required Dev Deps:
- [`streamline`](https://github.com/Sage/streamlinejs)
- [`babel-plugin-streamline`](https://github.com/Sage/babel-plugin-streamline)
- `babel-preset-es2015`
- When using React server-side:
	- `babel-preset-react`
- When using Gulp:
	- [`gulp`](http://gulpjs.com/)
	- [`gulp-babel`](https://github.com/babel/gulp-babel)
	- More complex shell command support, like for running a command on each file as part of a Gulp Task:
		- [`gulp-exec`](https://www.npmjs.com/package/gulp-exec) Note that for simply executing a command, they recommend just using `child_process/exec`.  That is what is done here.

Some quick start ideas:
- [`body-parser`](https://www.npmjs.com/package/body-parser) for easy HTTP body parsing. (note: _single-part bodies only!_  See project page for libraries for multipart bodies.)
- One of:
	- [`express`](http://expressjs.com/) for easy HTTP serving.
	- [`express-streamline`](https://github.com/aseemk/express-streamline) for easy HTTP serving with direct Streamline support.




Project Setup
-------------


### Directory Structure:

```
<project>/
	package.json
	source/
		index._js - Server entry point.
	bin/ - Contains files generated by the build step.
		index.js - Compiled server entry point.
```


### Gulpfile

```js
var exec = require( 'child_process' ).exec;

var gulp = require( 'gulp' );
var babel = require( 'gulp-babel' );

gulp.task( `default`, [ `build` ]);

gulp.task( `serve`, ( next ) => {
	const childProc = exec( `_node --runtime fibers source`, ( err, stdout, stderr ) => {
		console.log( stdout );
		console.warn( stderr );
	});

	childProc.stdout.on( `data`, data => console.log( `${ data }` ) );
	childProc.stderr.on( `data`, data => console.log( `${ data }` ) );

	return next();
});

gulp.task( `build`, () => {
	return gulp.src( 'source/**/*', { base: 'source' })
		.pipe( babel({
			presets: [ 'es2015' ],
			plugins: [
				[ 'streamline', { runtime: 'fibers' }]
			]
		}))
		.pipe( gulp.dest( 'bin' ) )
		;
});
```



Explanation
-----------


### Why wrap serve in a call to `child_process/exec` in a Gulpfile?

As of writing, for whatever reason, simply using `_node source` (any runtime) as an npm script in a project whose path contains a space results in an error that resembles the following:

```
# For example, project in /path/with/space in the name/

module.js:327
    throw err;
    ^

Error: Cannot find module '/path/with/space'
    at Function.Module._resolveFilename (module.js:325:15)
    at Function.Module._load (module.js:276:25)
    at Function.Module.runMain (module.js:429:10)
    at startup (node.js:139:18)
    at node.js:999:3
```

Strangely enough, running this using `child_process/exec` works just fine.  So that's why for now.  Something somewhere may not be quoting a string.


### `import` versus `require`

Because this is being run through Babel, you can use ES6 `import` syntax to grab node.js modules.  I don't know if this is recommended or not since you're technically writing to `require` calls, but it looks like `import`s.  Being able to write `export default <whatever>` is also nice, but is that worth the additional indirection?  Probably a question you need to answer on whatever project you're working on.
