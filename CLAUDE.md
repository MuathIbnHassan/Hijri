# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository scope

This is a **standalone legacy PHP library** (single class, no dependencies, no build) — it is *not* a Laravel project. The parent directory `/Users/m.ibnhassan/Development/tvtc-pt/CLAUDE.md` describes a Laravel app and **does not apply here**: ignore those Laravel/Filament/Pest/Pint guidelines when editing this repo. There is no Composer, no test runner, no Artisan, no build pipeline.

To run the demo: serve the directory with any PHP-capable web server (e.g. `php -S localhost:8000`) and open `example.php`. There is no other "build" or "test" command.

## Architecture

Everything lives in one class: `uCal` in `uCal.class.php`. Consumers `include "uCal.class.php"; $d = new uCal;` and call methods on the instance (see `example.php`).

The conversion is **table-driven**, not algorithmic. The class holds a precomputed lookup array `$jdl` of Modified Julian Date values — one entry per Hijri month — covering Hijri years 1300 onward (matching the official Umm al-Qura calendar used in Saudi Arabia). All conversions pivot through this table:

- `g2u($day, $month, $year)` — Gregorian → Hijri. Uses PHP's built-in `GregorianToJD()` to get a Julian Day, subtracts 2400000 to get MJD, then linearly scans `$jdl` for the bracketing Hijri month. Returns an array keyed by `month`, `day`, `year`, `ln` (Islamic lunation number), `ml` (month length).
- `u2g($day, $month, $year)` — Hijri → Gregorian. Indexes directly into `$jdl` (no scan), then converts MJD back via `JDToGregorian()`.
- `date($format, $timestamp = 0, $hijri = 1)` — A drop-in replacement for PHP's `date()` that emits Hijri output by default. It first calls native `date()` to extract Gregorian components and locale-independent fields, then for Hijri mode swaps in the converted day/month/year/month-name. The format-token substitution is implemented by replacing tokens with rare placeholder characters (`¢`, `£`, `ç`, `¥`, `©`, `ï`, `â`, `®`) so that a second pass can substitute the localized strings without re-matching `date()` tokens — **don't change those placeholders without auditing both `str_replace` calls**.
- `mktime($hour, $minute, $second, $month, $day, $year)` — Hijri equivalent of PHP `mktime()`; converts the Hijri date to Gregorian via `u2g()` and delegates to native `mktime()`.
- `setLang('en'|'ar')` — Toggles the localization arrays (`uF`, `uM`, `D`, `l`, `F`, `M`, `a`, `A`) used by `date()`. Arabic is the only non-English locale wired up.

The class depends on PHP's **calendar extension** (`GregorianToJD`, `JDToGregorian`). If those are unavailable, both conversion methods break.

## Things to be careful about when editing

- The file uses **PHP short open tags** (`<?` / `<?=`) and PHP 4-era syntax: `var $x` instead of visibility modifiers, and a constructor named `function uCal()` instead of `__construct()`. This will emit deprecation warnings on PHP 7+ and is broken on PHP 8 (PHP 4 constructors removed). Do not "modernize" syntax casually — the project's compatibility expectations are not documented, so confirm with the user before bumping syntax to a newer PHP baseline.
- Inside `g2u()` / `u2g()`, array keys are written as bareword constants (`$id[month]`, `$g[year]`) rather than strings (`$id['month']`). On modern PHP this only works because no constants of those names are defined; emits `E_WARNING`/fatals depending on version. Same caveat as above: tread carefully before "fixing".
- The `$jdl` table is finite — it covers a bounded range of Hijri months. Conversions beyond that range will silently produce wrong results (the `foreach` in `g2u` falls through, leaving `$i` at the last index). Don't add range validation without checking what existing callers expect.
- `example.php` interpolates `$_POST` values directly into output and into the conversion calls. It is a demo, not production-safe; do not copy its patterns into anything user-facing.
