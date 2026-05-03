# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository scope

This is a **standalone PHP library** (single class, no runtime dependencies) — it is *not* a Laravel project. The parent directory `/Users/m.ibnhassan/Development/tvtc-pt/CLAUDE.md` describes a Laravel app and **does not apply here**: ignore those Laravel/Filament/Pest/Pint guidelines when editing this repo. There is no Composer-installed dependency, no test runner, no Artisan, no build pipeline.

To run the demo: `composer install` once, then serve with any PHP-capable web server (e.g. `php -S localhost:8000`) and open `example.php`. There is no other "build" or "test" command.

## Layout

- `src/UCal.php` — the entire library: a single class `Hijri\UCal`. PSR-4 autoloaded.
- `example.php` — interactive demo (forms for Hijri↔Gregorian, formatted output samples).
- `composer.json` — declares `top7up/hijri`, `php >=7.0`, `ext-calendar`, PSR-4 `Hijri\\` → `src/`.
- `README.md`, `LICENSE`, `sample1.gif` — docs and assets.

## Architecture

Everything lives in one class: `Hijri\UCal` in `src/UCal.php`. Consumers do `require 'vendor/autoload.php'; $d = new \Hijri\UCal();` and call methods on the instance (see `example.php`).

The conversion is **table-driven**, not algorithmic. The class holds a precomputed lookup array `$jdl` of Modified Julian Date values — one entry per Hijri month — covering Hijri years 1300 onward (matching the official Umm al-Qura calendar used in Saudi Arabia). All conversions pivot through this table:

- `g2u($day, $month, $year)` — Gregorian → Hijri. Uses PHP's built-in `GregorianToJD()` to get a Julian Day, subtracts 2400000 to get MJD, then linearly scans `$jdl` for the bracketing Hijri month. Returns an array keyed by `'month'`, `'day'`, `'year'`, `'ln'` (Islamic lunation number), `'ml'` (month length).
- `u2g($day, $month, $year)` — Hijri → Gregorian. Indexes directly into `$jdl` (no scan), then converts MJD back via `JDToGregorian()`.
- `date($format, $timestamp = 0, $hijri = 1)` — A drop-in replacement for PHP's `date()` that emits Hijri output by default. It first calls native `date()` to extract Gregorian components and locale-independent fields, then for Hijri mode swaps in the converted day/month/year/month-name. The format-token substitution is implemented by replacing tokens with rare placeholder characters (`¢`, `£`, `ç`, `¥`, `©`, `ï`, `â`, `®`) so that a second pass can substitute the localized strings without re-matching `date()` tokens — **don't change those placeholders without auditing both `str_replace` calls**.
- `mktime($hour, $minute, $second, $month, $day, $year)` — Hijri equivalent of PHP `mktime()`; converts the Hijri date to Gregorian via `u2g()` and delegates to native `mktime()`.
- `setLang('en'|'ar')` — Toggles the localization arrays (`uF`, `uM`, `D`, `l`, `F`, `M`, `a`, `A`) used by `date()`. Arabic is the only non-English locale wired up.

The class depends on PHP's **calendar extension** (`GregorianToJD`, `JDToGregorian`). If unavailable, both conversion methods break — `composer.json` declares this as a hard requirement (`ext-calendar`).

## PHP-version policy

The intent of this library is to install on the **widest plausible range of PHP versions**. The Composer floor is `>=7.0`, and the code itself uses nothing newer than PHP 5.0 features. **Do not introduce newer-PHP syntax** in changes here:

- No scalar type hints, return types, nullable types, union/intersection types.
- No typed properties, readonly, enums, first-class callable syntax, named arguments.
- No arrow functions, null-safe operator, match expressions.
- Stick to `Array(...)` literals as written (changing to `[]` would still be valid on every supported version, but the author preference is minimal-diff).

If a PHP-feature change is genuinely needed, raise the floor in `composer.json` first, with explicit user approval.

## Editing notes

- The `$jdl` table (~890 entries) is the entire correctness payload. Touching it changes calendar output; treat it as data, not code.
- Inside `g2u()` and `u2g()`, array keys are quoted strings (`$id['month']`, `$g['year']`) — PHP 8 fatals on bareword keys. Don't undo that.
- The `date()` placeholder-character trick (`¢ £ ç ¥ © ï â ®`) is intentional and fragile. Don't change those characters without auditing both `str_replace` calls in the method.
- `example.php` interpolates `$_POST` values directly into output and into the conversion calls. It is a demo, not production-safe; do not copy its patterns into anything user-facing.
- The original copyright header at the top of `src/UCal.php` (Abdul-Aziz Al-Oraij, 2007, GPL) is intentionally preserved. The same author is credited in `composer.json`.
