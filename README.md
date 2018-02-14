# `Intl.DateTimeFormat.prototype.formatRange`

## Proposal \[Stage 0\]

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
the locale-specific tokens representing each parts of the formatted date range.

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
//   { type: 'hour',      value: '10'  },
//   { type: 'literal',   value: ':'   },
//   { type: 'minute',    value: '00'  },
//   { type: 'literal',   value: ' – ' },
//   { type: 'hour',      value: '11'  },
//   { type: 'literal',   value: ':'   },
//   { type: 'minute',    value: '00'  },
//   { type: 'literal',   value: ' '   },
//   { type: 'dayPeriod', value: 'AM'  }
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

*   Support for ranges can be added to other formatters (e.g Intl.NumberFormat)
    by simply providing a `.formatRange()` method, avoiding the need to create
    an additional range formatter for every regular formatter.
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
