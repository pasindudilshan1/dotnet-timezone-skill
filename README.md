# dotnet-timezone-skill

A Claude AI skill that resolves the correct timezone for any address, city, or document - and returns Windows ID, IANA ID, UTC offset, DST flag, and ready-to-use C# code.

## Installation

### Recommended (clone directly into Claude skills directory)

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/pasindudilshan1/dotnet-timezone-skill.git ~/.claude/skills/dotnet-timezone-skill
```

### Manual install (skill file only)

```bash
mkdir -p ~/.claude/skills/dotnet-timezone-skill
cp dotnet-timezone-skill/SKILL.md ~/.claude/skills/dotnet-timezone-skill/
```

### Install in Claude.ai (no CLI needed)

1. Download `releases/dotnet-timezone-skill.skill`
2. Go to **Claude.ai → Settings → Skills**
3. Upload the `.skill` file

## Usage

Paste any address, city, or document text:

```
What timezone is 42 Galle Road, Colombo, Sri Lanka?
```

Or ask directly:

```
timezone for Tokyo
what's the Windows TimeZoneInfo ID for Dubai?
convert UTC to Berlin time in C#
my app runs on Linux Docker - how do I safely get the local timezone?
```

Or paste a full document and Claude extracts all locations automatically:

```
Invoice from Nexgen Solutions, Colombo, Sri Lanka.
Ship to: 1600 Amphitheatre Pkwy, Mountain View, CA 94043, USA.
```

## Overview

.NET timezone handling is a well-known source of bugs - especially in cross-platform apps running on Linux or Docker, where Windows timezone IDs silently fail. This skill gives Claude the context to always return the right IDs, pick the correct library, and warn about the pitfalls.

Claude doesn't need a hardcoded database of every timezone. It reasons from trained geographic knowledge (IANA tz database docs, Windows timezone registry, OpenStreetMap, Wikipedia geography) to map any location to its correct timezone at query time. The skill's reference index anchors the most common zones for fast, consistent output.

## What Claude returns for every location

```
📍 Colombo, Sri Lanka
   Windows ID  : "Sri Lanka Standard Time"
   IANA ID     : "Asia/Colombo"
   UTC Offset  : +05:30 (no DST)

// Cross-platform - works on Windows + Linux
// NuGet: dotnet add package TimeZoneConverter
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTime local  = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

## Before / After examples

### Timezone lookup

| | Code |
|---|---|
| **Before** | `TimeZoneInfo tz = TimeZoneInfo.FindSystemTimeZoneById("Asia/Colombo");` |
| **After** | `TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");` |

`FindSystemTimeZoneById` with an IANA ID fails silently on Windows. `TZConvert` accepts both Windows and IANA IDs on any platform.

### DateTime storage

| | Code |
|---|---|
| **Before** | `entity.CreatedAt = DateTime.Now;` |
| **After** | `entity.CreatedAt = DateTime.UtcNow;` |

Storing local time in a database is a classic timezone bug. Always store UTC; convert at the presentation layer.

### API serialization

| | Code |
|---|---|
| **Before** | `public DateTime ScheduledAt { get; set; }` |
| **After** | `public DateTimeOffset ScheduledAt { get; set; }` |

`DateTime` loses the offset. `DateTimeOffset` carries it through serialization.

### Scheduling across timezones

| | Code |
|---|---|
| **Before** | `new DateTime(2024, 12, 1, 9, 0, 0)` |
| **After** | `TimeZoneInfo.ConvertTimeToUtc(scheduledLocal, TZConvert.GetTimeZoneInfo("Asia/Colombo"))` |

Scheduling "9am Colombo time" requires an explicit conversion to UTC before passing to your job scheduler.

## Full example

**Input (document with multiple addresses):**

> Invoice from Nexgen Solutions, 42 Galle Road, Colombo 03, Sri Lanka.  
> Ship to: 1600 Amphitheatre Pkwy, Mountain View, CA 94043, USA.  
> CC: Shinjuku office, Tokyo, Japan.

**Output from Claude:**

```
📍 Colombo, Sri Lanka
   Windows ID  : "Sri Lanka Standard Time"
   IANA ID     : "Asia/Colombo"
   UTC Offset  : +05:30 | DST: no

📍 Mountain View, CA, USA
   Windows ID  : "Pacific Standard Time"
   IANA ID     : "America/Los_Angeles"
   UTC Offset  : -08:00 / -07:00 | DST: yes

📍 Tokyo, Japan
   Windows ID  : "Tokyo Standard Time"
   IANA ID     : "Asia/Tokyo"
   UTC Offset  : +09:00 | DST: no

// NuGet: dotnet add package TimeZoneConverter
var timezones = new Dictionary<string, TimeZoneInfo>
{
    ["Colombo"]       = TZConvert.GetTimeZoneInfo("Asia/Colombo"),
    ["Mountain View"] = TZConvert.GetTimeZoneInfo("America/Los_Angeles"),
    ["Tokyo"]         = TZConvert.GetTimeZoneInfo("Asia/Tokyo"),
};

DateTime utcNow = DateTime.UtcNow;
foreach (var (place, tz) in timezones)
{
    DateTime local = TimeZoneInfo.ConvertTimeFromUtc(utcNow, tz);
    Console.WriteLine($"{place}: {local:yyyy-MM-dd HH:mm:ss} (UTC{tz.GetUtcOffset(utcNow):hh\\:mm})");
}
```

## 7 C# patterns covered

| # | Scenario | Library |
|---|---|---|
| 1 | Simple conversion, Windows-only app | `TimeZoneInfo` (built-in) |
| 2 | Cross-platform / Linux / Docker | `TimeZoneConverter` ← default recommendation |
| 3 | DST arithmetic, recurring schedules | `NodaTime` |
| 4 | API responses, database storage | `DateTimeOffset` |
| 5 | ASP.NET Core service layer | `TimeZoneConverter` + `DateTimeOffset` |
| 6 | Hangfire / Quartz recurring jobs | `TimeZoneConverter` + cron |
| 7 | DST ambiguous/invalid time handling | `NodaTime` strict mode |

## Common pitfalls Claude warns about

- `FindSystemTimeZoneById("Asia/Colombo")` **fails on Windows** - IANA IDs only work on Linux
- `FindSystemTimeZoneById("Sri Lanka Standard Time")` **fails on Linux** - Windows IDs only work on Windows
- `DateTime.Now` in server code stores the server's local time, not UTC
- `DateTimeKind.Unspecified` silently breaks comparisons across zones
- Azure App Service on Linux uses IANA IDs; on Windows it uses Windows IDs - same code, different behavior depending on hosting plan
- During DST transitions, one hour is either skipped or repeated - naive conversion produces wrong results

## Repository structure

```
dotnet-timezone-skill/
├── README.md
├── releases/
│   └── dotnet-timezone-skill.skill     ← install this in Claude.ai
└── dotnet-timezone-skill/
    ├── SKILL.md                        ← skill logic + address resolution
    └── references/
        ├── timezone-index.md           ← Windows ↔ IANA anchor table
        └── code-patterns.md            ← 7 C# patterns
```

## References

- [IANA Time Zone Database](https://www.iana.org/time-zones) - authoritative zone definitions
- [Windows Time Zone IDs](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/default-time-zones) - Microsoft docs
- [TimeZoneConverter NuGet](https://github.com/mattjohnsonpint/TimeZoneConverter) - by Matt Johnson-Pint
- [NodaTime](https://nodatime.org) - by Jon Skeet

## Version history

- **1.1.0** - Added address-to-timezone resolution; multi-location document extraction; combined C# snippet output
- **1.0.0** - Initial release: timezone ID lookup, 7 C# patterns, cross-platform guidance

## License

MIT
