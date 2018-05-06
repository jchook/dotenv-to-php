# dotenv-to-php

Convert dotenv files to PHP.

![Demo of dotenv-to-php](https://i.imgur.com/b7hMNpp.gif)


## Why use dotenv?

Storing configuration in the environment has many inherent advantages. For example, [it's a tenant of The Twelve Factor App](https://www.12factor.net/config) and allows you to keep your passwords separate from your code.

However, it's not always practical to set environment variables on development machines or continuous integration servers where multiple projects are run. In these scenarios, **dotenv** can load variables from a .env as needed.

To get started, create a .env file in the root directory of your project. Add all environment-specific variables (anything likely to change between servers or deployments) on individual lines like so:

```sh
# Application
APP_HOST="myapp.com"
APP_URI="https://$APP_HOST/"
APP_NAME="My Application"

# Database
DB_HOST=localhost
DB_USER=myapp
DB_PASS=mypass
```

Notice that you should quote values with spaces, etc. and that it's possible to reference previously defined variables as in normal shell environments. Comments beginning with `#` are ignored.

**Do not commit your .env file to source control.** Instead, store it somewhere secure, or use a tool like [vault](https://www.vaultproject.io/).


## Why use dotenv-to-php?

Instead of parsing a .env at runtime with complex dependencies, we can use `dotenv-to-php` to compile our .env to native PHP.

**It's fast.** The conversion is instantaneous. At runtime, simply load your env vars using the generated PHP. You can even use it in production.

**It's simple.** No runtime dependency. The transpiler is only [42 lines with comments](bin/dotenv-to-php), contains no regex, and no custom parsers. Contrast this with [phpdotenv](https://github.com/vlucas/phpdotenv/blob/475e5e0d27d669a59f9a6d04844255fa302d5d39/src/Loader.php#L228).


## Install

You can just [download the one file](bin/dotenv-to-php) **-OR-** add it as a dev dependency via composer:

	composer require --dev jchook/dotenv-to-php


In your PHP application, simply include the to-be-compiled `.env.php` file.

```php
<?php

// somewhere in index.php
include_once __DIR__ . '/.env.php';

?>
```


## Building `.env.php`

Ideally you can integrate the `.env.php` file generation into your existing build process (either for CI or local development). All examples below assume that your dotenv file is saved in the current working directory as `.env`.

### Bash

The simplest method is to call [`bin/dotenv-to-php`](bin/dotenv-to-php) directly.

```sh
/path/to/dotenv-to-php
```

Usage is roughly `dotenv-to-php [infile] [outfile]`. If no infile or outfile is specified, they default to `.env` and `$infile.php` respectively. You can also specify `-` (hyphen) for stdin or stdout... respectively.

### Composer

If you installed via composer, you can integrate it as a script in your `composer.json` file:

```json
{
	"scripts": {
		"build-env": "dotenv-to-php .env .env.php"
	}
}
```

Then invoke via:

```sh
composer build-env
```

### Node JS

Many folks are using node js to compile their static assets such as javascript. If you want to plug-in to this existing build process, you can use the built-in [child_process](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options) module.

```js
const childProcess = require('child_process');
childProcess.spawn('/path/to/dotenv-to-php', ['.env', '.env.php']);
```


### Webpack 4+

You can easily incorporate the above into an ad-hoc webpack plugin. In your `webpack.config.js` file:

```js
const childProcess = require('child_process');
module.exports = {
	
	// ... other config here ...
	
	plugins: [
		
		// ... other plugins here ...
		
		// Build PHP env
		{apply: (compiler) => {
			compiler.hooks.afterEnvironment.tap('DotEnvToPhp', (compilation) => {
				childProcess.spawn('/path/to/dotenv-to-php', ['.env', '.env.php']);
			});
		}}
	]
};
```



## License

MIT.