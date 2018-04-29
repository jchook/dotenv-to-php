# dotenv-to-php

Easily convert dotenv `.env` files to `.env.php`.


## Rationale

1. **It's faster.** Converting your dotenv to PHP allows it to be stored in opcache and avoids comparatively expensive disk I/O and parsing on every request.

2. **It's leaner.** No runtime code. It contains no regex, and no custom parsers. Contrast this with [phpdotenv](https://github.com/vlucas/phpdotenv/blob/475e5e0d27d669a59f9a6d04844255fa302d5d39/src/Loader.php#L228).


## Install

You can just [download the one file](bin/dot-env-to-php) **-OR-** add it as a dev dependency via composer:

	composer require --dev jchook/dotenv-to-php


In your PHP application, simply include the to-be-compiled `.env.php` file.

```php
<?php

// somewhere in index.php
include_once __DIR__ . '/.env.php';

?>
```


## Integrate

Ideally you can integrate the `.env.php` file generation into your existing build process (either for CI or local development). All examples below assume that your dotenv file is saved in the current working directory as `.env`.


### Shell

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


### Webpack

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