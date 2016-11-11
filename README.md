# Browser Test Session Module

[![Build Status](https://travis-ci.org/silverstripe-labs/silverstripe-testsession.svg)](https://travis-ci.org/silverstripe-labs/silverstripe-testsession)

## Overview

This module starts the PHP built-in webserver connected to a test database,
in order to test a SilverStripe application in a clean state.
Usually the session is started on a fresh database with only default records loaded.
Further data can be loaded from YAML fixtures or database dumps.

The session is persisted in a file which is generated upon starting the session.
As long as this file exists, the test session is considered in progress,
both in web browsers and command-line execution. By default, the file
is stored in the webroot under `assets/TESTS_RUNNING-<id>.json`. The `<id>` value
is a random token stored in the browser session, in order to make the
test session specific to the executing browser, and allow multiple
people using their own test session in the same webroot.

The module also serves as an initializer for the
[SilverStripe Behat Extension](https://github.com/silverstripe-labs/silverstripe-behat-extension/).
It is required for Behat because the Behat CLI test runner needs to persist
test configuration just for the tested browser connection,
available on arbitary URL endpoints. For example,
we're setting up a test mailer which writes every email
into a temporary database table for inspection by the CLI-based process.


## Usage via PHP

You can launch a new test server with the test server factory. By default, no fixture will be
loadedand it will use the first available port, starting its search at 8080:

```php
use SilverStripe\TestSession\TestServerFactory;

$factory = new TestServerFactory();
$server = $factory->launchServer();
```

`BASE_PATH` must be set as a PHP define. If not, you can pass a path as an argument to
TestServerFactory's constructor.

The object that is returned has a few helper methods, including:

 * `$server->getURL()` will give you the URL of your test site, in the form "http://localhost:8080".
 * `$server->stop()` will shut down the server

You can specify a number of options in a map passed to launch server:

```php
$server = $factory->launchServer([
    'host' => 'localhost',
    'preferredPort' => 8080,
    'fixtureFile' => 'tests/fixtures/something.yml',
    'bootstrapFile' => 'tests/bootstrap/something.php',
]);
```

 * **host:** The host to listen on. Defaults to `0.0.0.0` which means listen on every IP
 * **preferredPort:** The preferred port to start searching for a free port from. Defaults to 8080
 * **fixtureFile:** A YAML fixture file to load into the test database
 * **bootstrapFile:** A PHP script to run after the Composer autoloader is available and before
   `main.php` is called.
 * **requireDefaultRecords:** A boolean value. Set to true if you want the default records to be
   inlcuded.
 * **mailer:** An alternative class or service name to plug in as the SilverStripe mailer. Typically
   this is used to record emails instead of actually sending them.
 * **datetime:** Sets a simulated date used for all framework operations. Format as
   "yyyy-MM-dd HH:mm:ss" (Example: "2012-12-31 18:40:59").

## Usage via CLI

For manual test, runs the `test-serve` CLI script may be preferable:

```sh
$> vendor/bin/test-serve --fixture-file tests/fixtures/something.yml
```
The options `--host`, `--port`, `--fixture-file`, and `--bootstrap-file` do the same as the items
above.

In addition, there is an option, `--open`, that takes no value. If set, it will open the new server
in a web browser.

## Clearing the test session

The recommended way to clear the test session is to call `$server->stop()` and start a new server
with `$factory->launchServer()`. Most of the execution time is spent re-creating the database in anty
case; starting the server only adds about 20ms of overhead.

On the command line, kill the script and
re-execute `vendor/bin/test-serve`.

## Can I run test sessions on another webserver?

No, you can't. The previous version of the testsession module allowed this through sacrifices to
security which mean that accidentally deploying the testsession module would be a major security
vulnerability. We've opted to remove this.

If you need to run tests against a specific webserver back-end, you could do this by manually
manipulating the content of the database on a regular deployment, e.g. by loading sspak files into
the site. If you did this, you wouldn't need the testsession module at all.

However, we recommend that the bulk of your testing work is made agnostic of the webserver that the
code runs on.
