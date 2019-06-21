## Intro
Gulp is used for a variety of tasks, but the most common usage for us will likely be to compile SCSS files to CSS and to combine and minify JS files. Gulp can also watch for changes in files so that as you edit files, the changes will be picked up and included in the built files, meaning that you don't need to manually run the Gulp task again.

## Setup

To add Gulp to a project, you need to install it via npm or yarn. Use `npm install gulp` or `yarn add gulp` in the root directory of your project to install gulp and its dependencies. To do anything, gulp requires other gulp packages to perform most of the things that you will want it to do.

For example, if you wanted to transpile ES6 JavaScript down to ES5, you would need to use the package `gulp-babel` to perform this task.

### Packages

- gulp-babel - Transpile ES6 to ES5
- gulp-uglify-es - minifies JS files
- gulp-concat - combines JS files
- gulp-sass - compiles .sscs files to .css files
- gulp-clean-css - optimises css
- gulp-rename - renames files

Go to <a href="https://www.npmjs.com">npmjs.com</a> to find packages that do what you need.

### Gulpfile

Once you have installed the packages via the terminal, you need to create a Gulpfile. This is a file named `gulpfile.js` which lives in the directory of your project that contains all of the files and subdirectories that your gulpfile needs to affect.

In your Gulpfile, import the packages individually and assign them to variables:

```
const gulp = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify-es').default;
const concat = require('gulp-concat');
const sass = require('gulp-sass');
const cleanCSS = require('gulp-clean-css');
const rename = require('gulp-rename');
```

N.B. for the package `gulp-uglify-es`, you need to add '.default' to the require statement.

## Creating Functions

Now that you have imported the packages that you need, you can create functions to perform the various tasks you need. You can define these just like normal functions in JavaScript:

```
const inputJS = [
	'src/scripts/app/index.js',
	'src/scripts/header/index.js',
];

function buildJS() {
	return gulp.src(inputJS)
		.pipe(concat('all.js'))
		.pipe(babel({
			presets: ['@babel/env']
		}))
		.pipe(rename('build.js'))
		.pipe(gulp.dest('src/scripts/build'))
}
```

In this example, we define an array that contains the paths to some JavaScript files that we want to combine and transpile to ES5. The function pipes the files into `concat` to combine them, `babel` to transpile them, `rename` to set the name to 'build.js', and `gulp.dest` to create the new file. If the file already exists, it will be overwritten.

Gulp often crashes if it finds syntax errors in CSS files. To solve this, include `.on('error', callback)` to the function that is dealing with the CSS.

```
const inputSCSS = 'src/styles/index.scss';
function buildCSS() {
	return gulp.src(inputSCSS)
		.pipe(sass())
		.on('error', function(error) {
			console.log(error.toString());
			this.emit('end');
		})
		.pipe(cleanCSS())
		.pipe(rename('build.min.css'))
		.pipe(gulp.dest('src/styles/build'))
}
```

In this example, the function that is passed to `.on` logs the error to the console then calls `this.emit('end')`. It is important to emit 'end' or the error will be logged, but the task will not continue, resulting in a crash anyway.

## Gulp 4: parallel and series

Gulp 4 introduced the `parallel` and `series` methods, which were not present in Gulp < 4. These methods can are used to run the functions that you have defined in your Gulpfile. Parallel, as the name suggests, runs the functions at the same time, meaning that if you have 4 functions that each take about 2 seconds to complete, you can complete all of them together in 2 seconds instead of 8. Series, similarly, runs tasks one after another. These two functions can be nested within each other to arbitrary depth.

```
gulp.parallel(gulp.series(buildJS, uglifyJS), buildCSS);
```
In this example, the JS tasks and the CSS task are run in parallel, meaning that they run at the same time, but the JS tasks run one after each other, as the CSS is running separately. Importantly, the JavaScript tasks need to run in series, because the uglifyJS task relies on the buildJS task already having been completed.

## Using tasks in the terminal

To use a task in the terminal, it needs to be exported from your Gulpfile.

```
exports.watch = watch;
exports.build = build;
```
In this example, we are referencing 2 function that would have been created earlier in the file, and exporting them with the same names as they are defined with. In the terminal, we can now run the commands `gulp watch` and `gulp build` to make these functions run.

If you want to export something by default, and not have it named anything, you can include this:
```
exports.default = watch
```
This means that if you run the command `gulp` in the terminal, it will choose the default task. This can be done in tandem with named exports, meaning you can have some named exports in addition to a default export.

## Example file

```
const gulp = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify-es').default;
const concat = require('gulp-concat');
const sass = require('gulp-sass');
const cleanCSS = require('gulp-clean-css');
const rename = require('gulp-rename');

const inputJS = [
	'src/scripts/app/index.js',
	'src/scripts/header/index.js',
];
const inputSCSS = 'src/styles/index.scss';
const watchSCSS = 'src/styles/**/*.scss';

function buildJS() {
	return gulp.src(inputJS)
		.pipe(concat('all.js'))
		.pipe(babel({
			presets: ['@babel/env']
		}))
		.pipe(rename('build.min.js'))
		.pipe(gulp.dest('src/scripts/build'))
}

function uglifyJS() {
	return gulp.src('src/scripts/build/build.min.js')
		.pipe(uglify())
		.pipe(gulp.dest('src/scripts/build'))
}

function buildCSS() {
	return gulp.src(inputSCSS)
		.pipe(sass())
		.on('error', function(error) {
			console.log(error.toString());
			this.emit('end');
		})
		.pipe(cleanCSS())
		.pipe(rename('build.min.css'))
		.pipe(gulp.dest('src/styles/build'))
}

function watch() {
	gulp.watch(inputJS, buildJS);
	gulp.watch(watchSCSS, buildCSS);
}

exports.build = gulp.parallel(gulp.series(buildJS, uglifyJS), buildCSS);
exports.default = watch;
```