Rector is using static reflection to load code without running it since version 0.10. That means your classes are found **without composer autoload and without running them**. Rector will find them and work with them as you have PSR-4 autoload properly setup. This comes very useful in legacy projects or projects with custom autoload.

Do you want to know more about it? Continue here:

- [From Doctrine Annotations Parser to Static Reflection](https://getrector.com/blog/from-doctrine-annotations-parser-to-static-reflection)
- [Legacy Refactoring made Easy with Static Reflection](https://getrector.com/blog/2021/03/15/legacy-refactoring-made-easy-with-static-reflection)
- [Zero Config Analysis with Static Reflection](https://phpstan.org/blog/zero-config-analysis-with-static-reflection) - from PHPStan

<br>

```php
use Rector\Config\RectorConfig;

return RectorConfig::configure()
    ->withAutoloadPaths([
        // discover specific file
        __DIR__ . '/file-with-functions.php',
        // or full directory
        __DIR__ . '/project-without-composer',
    ]);
```

## Include Files

Do you need to include constants, class aliases or custom autoloader? Use bootstrap files:


```php
use Rector\Config\RectorConfig;

return RectorConfig::configure()
    ->withBootstrapFiles([
        __DIR__ . '/constants.php',
        __DIR__ . '/project/special/autoload.php',
    ]);
```

Listed files will be executed like:

```php
include $filePath;
```

## Dealing with "Class ... was not found while trying to analyse it..."

Sometimes you may encounter this error ([see here for an example](https://github.com/rectorphp/rector/issues/6688)) even if the class is there and it seems to work properly with other tools (e.g. PHPStan).

In this case you may want to try one of the following solutions:

### Register

```php
use Rector\Config\RectorConfig;

return RectorConfig::configure()
    ->withAutoloadPaths([
        // the path to the exact class file
        __DIR__ . '/vendor/acme/my-custom-dependency/src/Your/Own/Namespace/TheAffectedClass.php',
        // or you can specify a wider scope
        __DIR__ . '/vendor/acme/my-custom-dependency/src',
        // WARNING: beware of performances, try to narrow down the path
        //          as much as you can or you will slow down each run
    ]);
```

### Register the path of the class to composer.json's `"files"` config, eg:

```json
{
    "autoload-dev": {
        "files": [
            "vendor/acme/my-custom-dependency/src/Your/Own/Namespace/TheAffectedClass.php"
        ]
    }
}
```

After that, dump classes and re-run Rector:

```bash
composer dump-autoload
```
