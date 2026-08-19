# Package Maven Standards

The exact PHPCS and PHPStan configurations that [Package Maven](https://package-maven.com)
uses when testing Magento 2 modules. Published so module authors can reproduce the
results locally.

## Why a relaxed ruleset?

Magento's out-of-the-box coding standard is a great baseline, but it is sometimes
a bit behind: some sniffs don't work properly in practice, and others have gaps
with newer PHP features (for example, requiring `@param` docblocks that only
duplicate native parameter, return and property types). This repository is a
deliberately relaxed version of that standard, tuned for Package Maven's
automated module testing — and it is shared here to be fully transparent about
which checks actually run and why each excluded one was dropped. The reasoning
for every exclusion is documented directly in the config files.

Contributions are very welcome. If you think any rule should be added back,
removed, or adjusted, please open an issue or send a pull request — the
templates will ask you to explain the intention and reasoning behind the change
so it can be discussed properly.

## Contents

| File | Purpose |
|------|---------|
| `phpcs/phpcs.xml` | PHPCS ruleset — the stock `Magento2` standard from [magento/magento-coding-standard](https://github.com/magento/magento-coding-standard) with a small set of exclusions (docblock formality, a few noisy/broken sniffs) |
| `phpstan/phpstan.neon` | PHPStan configuration — no rule exclusions except `missingType.iterableValue`, Test directories excluded |

## How Package Maven runs them

Each release is installed with `composer require --no-scripts --no-plugins` into a
clean Magento 2.4.9 project (PHP 8.5, locked dependencies), then analysed as
published on Packagist — only files in the composer dist archive are scanned:

```bash
vendor/bin/phpcs --report=junit --standard=phpcs.xml -q vendor/<vendor>/<package>

# levels 0–9, cumulative; the score is the highest level passing clean
vendor/bin/phpstan analyze -c phpstan.neon vendor/<vendor>/<package> --level=<N> --no-progress
```

PHPStan runs with `phpstan/extension-installer` and `bitexpert/phpstan-magento`
inside the full Magento installation so core classes resolve.

## Using it in your own project

```bash
composer require --dev package-maven/magento-standards magento/magento-coding-standard squizlabs/php_codesniffer phpstan/phpstan
```

```bash
vendor/bin/phpcs --standard=vendor/package-maven/magento-standards/phpcs/phpcs.xml app/code/Your/Module
vendor/bin/phpstan analyze -c vendor/package-maven/magento-standards/phpstan/phpstan.neon app/code/Your/Module --level=6
```
