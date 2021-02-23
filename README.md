# `Intl.DateTimeFormat.prototype.formatRange`

## Overview

### Motivation

It's common for websites to display date intervals or date ranges to show the
span of an event, such as a hotel reservation, the billing period of a
service, or other similar uses. In order to implement this, websites often
use localization libraries, such as Google Closure, to format the date range,
or they may simply resort to formatting both dates independently.

If following the second alternative, web developers may encounter problems such
as repeating fields that are common between the two dates, inappropriate order
of the dates for the locale or using an incorrect delimiter for the locale.

For example:

```javascript
let date1 = new Date(Date.UTC(2007, 0, 10)); // "Jan 10, 2007"
let date2 = new Date(Date.UTC(2007, 0, 20)); // "Jan 20, 2007"

let fmt = new Intl.DateTimeFormat('en', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
});

// The second date's 'month' and 'year' calendar fields are redundant, only
// 'day' provides new information.
console.log(`${fmt.format(date1)} – ${fmt.format(date2)}`);
// > 'January 10, 2007 – January 20, 2007'
```

Formatting date ranges in a concise and locale-aware way requires a
non-negligible amount of raw or compiled data: localized patterns are needed for
all the possible combinations of displayed calendar fields (e.g. `month`, `day`,
`hour`), their lengths (e.g. `2-digit`, `numeric`) and which is the largest
different calendar field between the range's two dates.

For example, formatting the following ranges using the same options (`{year:
'numeric', month: 'short', day: 'numeric'}`) requires two different patterns:

Date range from **January 10, 2017** to **January 20, 2017**:

* `'Jan 10 – 20, 2007'`
* Largest different calendar field: `'day'`

Date range from **January 10, 2017** to **February 20, 2017**:

* `'Jan 10 – Feb 20, 2007'`
* Largest different calendar field: `'month'`

Bringing this functionality into the platform will improve performance of the
web and developer productivity as they will be able to format date ranges
correctly, flexibly and concisely without the extra locale data size and
overhead.

### Status

**Stage 4**

* Initial Discussion: https://github.com/tc39/ecma402/issues/188
* PR: https://github.com/tc39/ecma402/pull/532 (merged)

Polyfill: [@formatjs/intl-datetimeformat](https://www.npmjs.com/package/@formatjs/intl-datetimeformat)

## Proposal

Add `formatRange(date1, date2)` and `formatRangeToParts(date1, date2)` to
`Intl.DateTimeFormat` to enable date range formatting.

This proposal is based on the ICU Date Interval Formatter and on the Unicode
CLDR TR-35 spec on date intervals.

*   http://icu-project.org/apiref/icu4j/com/ibm/icu/text/DateIntervalFormat.html
*   https://unicode.org/reports/tr35/tr35-dates.html#intervalFormats

### API

#### `Intl.DateTimeFormat.prototype.formatRange(date1, date2)`

This method receives two `Dates` and formats the date range in the most concise
way based on the `locale` and `options` provided when instantiating
`Intl.DateTimeFormat`.

#### `Intl.DateTimeFormat.prototype.formatRangeToParts(date1, date2)`

This method receives two `Dates` and returns an `Array` of objects containing
the locale-specific tokens representing each part of the formatted date range.

See:
[`Intl.DateTimeFormat.prototype.formatToParts`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat/formatToParts).

### Example Usage

#### `Intl.DateTimeFormat.prototype.formatRange(date1, date2)`

```javascript
let date1 = new Date(Date.UTC(2007, 0, 10, 10, 0, 0));
let date2 = new Date(Date.UTC(2007, 0, 10, 11, 0, 0));
let date3 = new Date(Date.UTC(2007, 0, 20, 10, 0, 0));
// > 'Wed, 10 Jan 2007 10:00:00 GMT'
// > 'Wed, 10 Jan 2007 11:00:00 GMT'
// > 'Sat, 20 Jan 2007 10:00:00 GMT'

let fmt1 = new Intl.DateTimeFormat("en", {
    year: '2-digit',
    month: 'numeric',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric'
});
console.log(fmt1.format(date1));
console.log(fmt1.formatRange(date1, date2));
console.log(fmt1.formatRange(date1, date3));
// > '1/10/07, 10:00 AM'
// > '1/10/07, 10:00 – 11:00 AM'
// > '1/10/07, 10:00 AM – 1/20/07, 10:00 AM'

let fmt2 = new Intl.DateTimeFormat("en", {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
});
console.log(fmt2.format(date1));
console.log(fmt2.formatRange(date1, date2));
console.log(fmt2.formatRange(date1, date3));
// > 'Jan 10, 2007'
// > 'Jan 10, 2007'
// > 'Jan 10 – 20, 2007'
```

#### `Intl.DateTimeFormat.prototype.formatRangeToParts(date1, date2)`

```javascript
let date1 = new Date(Date.UTC(2007, 0, 10, 10, 0, 0));
let date2 = new Date(Date.UTC(2007, 0, 10, 11, 0, 0));
// > 'Wed, 10 Jan 2007 10:00:00 GMT'
// > 'Wed, 10 Jan 2007 11:00:00 GMT'

let fmt = new Intl.DateTimeFormat("en", {
    hour: 'numeric',
    minute: 'numeric'
});

console.log(fmt.formatRange(date1, date2));
// > '10:00 – 11:00 AM'

fmt.formatRangeToParts(date1, date2);
// return value:
// [
//   { type: 'hour',      value: '10',  source: "startRange" },
//   { type: 'literal',   value: ':',   source: "startRange" },
//   { type: 'minute',    value: '00',  source: "startRange" },
//   { type: 'literal',   value: ' – ', source: "shared"     },
//   { type: 'hour',      value: '11',  source: "endRange"   },
//   { type: 'literal',   value: ':',   source: "endRange"   },
//   { type: 'minute',    value: '00',  source: "endRange"   },
//   { type: 'literal',   value: ' ',   source: "shared"     },
//   { type: 'dayPeriod', value: 'AM',  source: "shared"     }
// ]
```

## Alternatives

### `Intl.DateIntervalFormat`

Instead of adding `.formatRange()` and `.formatRangeToParts()` to
`Intl.DateTimeFormat`, a separate date interval formatter can be added. The API
would be the following:

*   `new Intl.DateIntervalFormat(locale, options)`: `options` are the same
    options currently used in `Intl.DateTimeFormat`.
*   `Intl.DateIntervalFormat.prototype.format(date1, date2)`
*   `Intl.DateIntervalFormat.prototype.formatToParts(date1, date2)`

Benefits:

*   Consistent with other I18n libraries such as ICU, which provide a separate
    date interval formatter.

Drawbacks:

*   Support for ranges can be added to other formatters (e.g
    `Intl.NumberFormat`) by simply providing a `.formatRange()` method, avoiding
    the need to create an additional range formatter for every regular
    formatter.
*   The `options` used to instantiate a `Intl.DateIntervalFormat` are the same
    ones used for
    [`Intl.DateTimeFormat`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat).
    Any options added to `Intl.DateTimeFormat` will need to be replicated in
    `Intl.DateIntervalFormat`.

## Prior art

*   ICU
    [com.ibm.icu.text.DateIntervalFormat](http://icu-project.org/apiref/icu4j/com/ibm/icu/text/DateIntervalFormat.html)
*   Apple Foundation
    [NSDateIntervalFormatter](https://developer.apple.com/documentation/foundation/nsdateintervalformatter)
*   Closure
    [goog.i18n.DateIntervalFormat](https://google.github.io/closure-library/api/goog.i18n.DateIntervalFormat.html)
