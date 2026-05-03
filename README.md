# Hijri

A small PHP library for converting dates between the Hijri (Umm al-Qura) and Gregorian calendars. Originally written by [Abdul-Aziz Al-Oraij](http://aziz.oraij.com/) in 2007; modernized for PHP 7.0+ / 8.x and Composer.

Features:

* Convert dates between Hijri and Gregorian calendars and vice-versa
* 100% compatibility with Umm al-Qura
* Returns dates formatted like the PHP `date()` function
* Outputs English/Arabic formatted Hijri and Gregorian dates and times
* Provides `date()` and `mktime()` function versions for Hijri

## Requirements

* PHP 7.0 or newer
* PHP `calendar` extension (used by `g2u` / `u2g` via `GregorianToJD` / `JDToGregorian`)

## Installation

```
composer require top7up/hijri
```

## Usage

```php
require __DIR__ . '/vendor/autoload.php';

$d = new \Hijri\UCal();

// Gregorian -> Hijri
$hijri = $d->g2u(22, 10, 2017);
// $hijri => ['month' => 2, 'day' => 2, 'year' => 1439, 'ln' => ..., 'ml' => ...]

// Hijri -> Gregorian
$greg = $d->u2g(20, 2, 1396);

// Formatted current Hijri date
echo $d->date('d/m/Y - l, F jS');

// Arabic localization
$d->setLang('ar');
echo $d->date('d/m/Y - l, F jS');

// Hijri mktime: build a Unix timestamp from a Hijri date
$ts = $d->mktime(22, 10, 30, 2, 3, 1428);
echo $d->date('d/m/Y H:i', $ts);          // Hijri
echo $d->date('d/m/Y H:i', $ts, 0);       // Gregorian (third arg = 0)
```

See `example.php` for a full demo (run with `php -S localhost:8000` after `composer install`).

## License

GPL-2.0-or-later. See `LICENSE`.
