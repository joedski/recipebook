Clientside: Browserify with Streamline and Babel
================================================

> node.js version as of writing this: `5.4.1`.  This recipe uses Gulp as the build system/taskrunner.

Required Deps:
- `streamline-runtime`

Required Dev Deps:
- [`browserify`](http://browserify.org/)
- [`babelify`](https://github.com/babel/babelify)
- [`babel-plugin-streamline`](https://github.com/Sage/babel-plugin-streamline)
- `babel-preset-es2015`
- `babel-preset-react`

Dev Deps when using Gulp:
- [`gulp`](http://gulpjs.com/)
- [`vinyl-buffer`](https://github.com/hughsk/vinyl-buffer)
- [`vinyl-source-stream`](https://github.com/hughsk/vinyl-source-stream)

Optional Dev Deps when using Gulp:
- [`gulp-sourcemaps`](https://github.com/floridoo/gulp-sourcemaps)
- [`gulp-if`](https://github.com/robrich/gulp-if)
- [`gulp-uglify`](https://github.com/terinjokes/gulp-uglify)
- [`streamqueue`](https://github.com/nfroidure/StreamQueue) - Useful if trying to bundle vendor scripts in the same file as the browserified app.


Project Setup
-------------

> Note: Giving your streamline-based files the `._js` extension is not necessary, but may be a helpful warning to anyone coming to the project.  Or it may just be confusing if they don't know about streamline.  Be sure to mention somewhere that the project uses streamline to hopefully reduce this confusion.  (Of course, then you must solve the project of getting people to RTFM...  Well.  Ignorance of the law does not mean immuity from it.)

The Browserify portion is based on [one of the recipes provided by the Gulp team](https://github.com/gulpjs/gulp/blob/master/docs/recipes/browserify-uglify-sourcemap.md).


### Directory Structure:

```
<project>/
	package.json
	client/
		index._js
		...
	public/
		(client side files end up here.)
```


### Gulpfile: Minimal/Least Fancy

```js
var gulp = require( 'gulp' );
var babel = require( 'gulp-babel' );
var source = require( 'vinyl-source-stream' );
var buffer = require( 'vinyl-buffer' );
var browserify = require( 'browserify' );
var babelify = require( 'babelify' );

gulp.task( 'build-clientside-scripts-app', () => {
	return browserify( 'client/index._js', { debug: true })
		.transform( babelify, {
			presets: [ 'es2015', 'react' ],
			plugins: [
				[ 'streamline', {
					// 'generators' will result in a faster product, but no support for old browsers.
					// Good for writing UI portion nw.js or electron apps.
					// Use 'fibers' for non-UI portion of those.
					runtime: 'callbacks'
					sourceMap: true
				}]
			]
		})
		.bundle()
		.on( 'error', ( err ) => { console.error( err ); this.emit( 'end' ); })
		.pipe( source( 'app-index.js' ) )
		.pipe( buffer() )
		.pipe( concat( 'app.js' ) )
		.pipe( gulp.dest( 'public' ) )
		;
});
```


### Gulpfile: Source Maps, Uglification, Single File

```js
var gulp = require( 'gulp' );
var gulpIf = require( 'gulp-if' );
var concat = require( 'gulp-concat' );
var sourcemaps = require( 'gulp-sourcemaps' );
var uglify = require( 'gulp-uglify' );
var babel = require( 'gulp-babel' );
var source = require( 'vinyl-source-stream' );
var buffer = require( 'vinyl-buffer' );
var browserify = require( 'browserify' );
var babelify = require( 'babelify' );
var streamqueue = require( 'streamqueue' );

const PRODUCTION = (gutil.env.type == 'production');

gulp.task( 'build-clientside-scripts', () => {
	var libs = gulp.src( 'client/lib/**/*' )
		.pipe( sourcemaps.init() )
		.pipe( babel({
			presets: [ 'es2015' ]
		}) )
		;

	var appBundler = browserify( 'client/index._js', { debug: true })
		.transform( babelify, {
			presets: [ 'es2015', 'react' ],
			plugins: [
				[ 'streamline', {
					runtime: 'callbacks',
					sourceMap: true
				}]
			]
		})
		;

	var app = appBundler
		.bundle()
		.on( 'error', ( err ) => { console.error( err ); this.emit( 'end' ); })
		.pipe( source( 'app-index.js' ) )
		.pipe( buffer() )
		.pipe( sourcemaps.init({ loadSourcemaps: true }) )
		;

	return streamqueue({ objectMode: true }, libs, app )
		.pipe( concat( 'app.js' ) )
		.pipe( gulpIf( PRODUCTION, uglify() ) )
		.pipe( sourcemaps.write( './' ) )
		.pipe( gulp.dest( 'public' ) )
		;
});
```
