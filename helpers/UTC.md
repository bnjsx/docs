# Bnjsx UTC

This module is designed for handling date and time manipulation in UTC (Coordinated Universal Time). With several functions for working with date and time strings in `YYYY-MM-DD hh:mm:ss` format, including parsing, formatting, and modifying date and time values.

## Table of Contents

1. **[Usage](#usage)**
   - **[Get Methods](#1-get-methods)**
   - **[Set Methods](#2-set-methods)**
   - **[Future Methods](#3-future-methods)**
   - **[Past Methods](#4-past-methods)**
   - **[Parsing Methods](#5-parsing-methods)**
   - **[Convert Dates](#6-convert-dates)**
   - **[Format Dates](#7-format-dates)**

## Usage

The `UTC` class is a utility class that provides a set of static methods to interact with UTC (Coordinated Universal Time) dates and times. It includes methods to retrieve, manipulate, and format dates and times in UTC, making it easier to handle date/time operations that are time zone-agnostic. Here’s a detailed explanation of how to use the `UTC` class:

### **1. Get Methods**

These methods allow you to retrieve various components of the current or the given UTC datetime.

```js
// Get the current UTC date
const currentDate = UTC.get.date();

// Get the current UTC time
const currentTime = UTC.get.time();

// Get the current UTC year
const currentYear = UTC.get.year();

// Get the current UTC month
const currentMonth = UTC.get.month();

// Get the current UTC hour
const currentHour = UTC.get.hour();

// Get the current UTC minute
const currentMinute = UTC.get.minute();

// Get the current UTC second
const currentSecond = UTC.get.second();

// Get the current UTC date and time
const currentDateTime = UTC.get.datetime();
```

### **2. Set Methods**

These methods allow you to modify specific components of the current or the given UTC datetime.

```js
// Set the current UTC year to 2025
console.log(UTC.set.year(2025));

// Set the current UTC month to March
console.log(UTC.set.month(3));

// Set the current UTC day to 15
console.log(UTC.set.day(15));

// Set the current UTC hour to 10
console.log(UTC.set.hour(10));

// Set the current UTC minute to 30
console.log(UTC.set.minute(30));

// Set the current UTC second to 50
console.log(UTC.set.second(50));
```

### **3. Future Methods**

These methods allow you to calculate future dates by adding a specific amount of time to the current or the given UTC datetime.

```js
// Add 2 years to the current UTC date
const futureYear = UTC.future.year(2);

// Add 3 months to the current UTC date
const futureMonth = UTC.future.month(3);

// Add 5 days to the current UTC date
const futureDay = UTC.future.day(5);

// Add 10 hours to the current UTC time
const futureHour = UTC.future.hour(10);

// Add 30 minute to the current UTC time
const futureMinute = UTC.future.minute(30);

// Add 50 second to the current UTC time
const futureSecond = UTC.future.second(50);
```

### **4. Past Methods**

These methods allow you to calculate past dates by subtracting a specific amount of time from the current or the given UTC datetime.

```js
// Subtract 2 years from the current UTC date
const pastYear = UTC.past.year(2);

// Subtract 3 months from the current UTC date
const pastMonth = UTC.past.month(3);

// Subtract 5 days from the current UTC date
const pastDay = UTC.past.day(5);

// Subtract 10 hours from the current UTC time
const pasthour = UTC.past.hour(10);

// Subtract 30 minutes from the current UTC time
const pastMinute = UTC.past.minute(30);

// Subtract 50 seconds from the current UTC time
const pastSecond = UTC.past.second(50);
```

### **5. Parsing Methods**

Extract the date part from a datetime string `YYYY-MM-DD`.

```js
console.log(UTC.parseDate('2024-12-15 00:00:00')); // 2024-12-15
```

Extract the time part from a datetime string `hh:mm:ss`.

```js
console.log(UTC.parseTime('2024-12-15 00:00:00')); // 00:00:00
```

### **6. Convert Dates**

Convert any datetime string from a specific timezone to UTC.

```js
// Convert date in Europe/London to UTC
console.log(UTC.from.date('2024-06-15 12:00:00', 'Europe/London'));
// Output: '2024-06-15 11:00:00' (1-hour difference)

// Convert date in Australia/Sydney to UTC
console.log(UTC.from.date('2024-12-25 15:00:00', 'Australia/Sydney'));
// Output: '2024-12-25 04:00:00' (11-hour difference from UTC)

// Convert date in America/New_York to UTC
console.log(UTC.from.date('2024-02-09 13:45:30', 'America/New_York'));
// Output: '2024-02-09 18:45:30' (5-hour difference)
```

### **7. Format Dates**

Convert your Date object to a string in the format `YYYY-MM-DD hh:mm:ss`.

```js
const formattedDate = UTC.formatDate(new Date());
```
