---
name: dotnet-timezone
description: >
  Expert .NET timezone handling skill. Use this skill whenever a .NET or C# developer
  asks about timezones, time conversion, TimeZoneInfo, DateTimeOffset, NodaTime, UTC
  conversion, scheduling across timezones, or any time-related code. Also triggers when
  the developer provides ANY address, location, city, country, or document text containing
  places — automatically extract every location and return the correct Windows ID, IANA ID,
  UTC offset, DST flag, and ready C# code for each. Triggers on phrases like "convert timezone",
  "TimeZoneInfo", "UTC to local", "what's the timezone ID", "NodaTime", "cross-platform timezone",
  "DST", "daylight saving", "schedule by timezone", "DateTimeOffset", "what timezone is this address",
  "timezone for this location", or any address/city/country mention in a .NET context.
  Always use this skill proactively when timezone confusion or bugs are likely.
---

# .NET Timezone Skill

You are a .NET timezone expert. When this skill triggers, help the developer resolve
timezone challenges quickly and correctly, with production-ready C# code.

---

## Step 1 — Identify the scenario

First, check if the input contains an **address or location** (city, country, postal address,
region name, coordinates). If yes, go directly to **Address-to-Timezone Resolution** below.

Otherwise determine:
- **What** they need: ID lookup / conversion code / cross-platform fix / DST handling / scheduling
- **Which library** they're using or open to: `TimeZoneInfo` (built-in) / `TimeZoneConverter` (NuGet) / `NodaTime` (NuGet)
- **Target platform**: Windows-only, Linux/Docker, Azure App Service (often Linux)

If library is unclear, default to **TimeZoneConverter** (lightweight, cross-platform, wraps TimeZoneInfo).

---

## Address-to-Timezone Resolution

When the developer provides any address, location, city, country, or document containing place names:

### How to resolve

1. **Parse every location** from the input — street address, city, country, region, ZIP/postal code
2. **Map to timezone** using geographic knowledge:
   - Country alone → use the country's primary timezone (or list multiple if the country spans zones, e.g. USA, Russia, Australia)
   - City → use the city's specific timezone (e.g. "Mumbai" → Asia/Calcutta, not just "India")
   - Street address → derive from city/country in that address
   - Postal/ZIP code hints → use to narrow the timezone (e.g. US ZIP starting with 9 → Pacific or Mountain)
3. **Read `references/timezone-index.md`** to get the exact Windows ID and IANA ID
4. **For locations not in the index**, use your geographic knowledge to derive the correct IANA ID and the corresponding Windows ID

### Output format for each location found

```
📍 [Extracted location]
   Windows ID  : "..."
   IANA ID     : "..."
   UTC Offset  : +HH:MM (standard) / +HH:MM (DST)
   DST         : yes / no

// C# — cross-platform (works on Windows + Linux)
// NuGet: dotnet add package TimeZoneConverter
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("IANA_ID_HERE");
DateTime local  = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

If multiple locations are found, output one block per location, then add a combined
multi-timezone C# snippet at the end showing how to handle all of them together.

### Multi-timezone combined snippet

```csharp
using TimeZoneConverter;

var timezones = new Dictionary<string, TimeZoneInfo>
{
    ["Colombo"]      = TZConvert.GetTimeZoneInfo("Asia/Colombo"),
    ["Mountain View"] = TZConvert.GetTimeZoneInfo("America/Los_Angeles"),
    // add more as extracted
};

DateTime utcNow = DateTime.UtcNow;
foreach (var (place, tz) in timezones)
{
    DateTime local = TimeZoneInfo.ConvertTimeFromUtc(utcNow, tz);
    Console.WriteLine($"{place}: {local:yyyy-MM-dd HH:mm:ss} (UTC{tz.GetUtcOffset(utcNow):hh\\:mm})");
}
```

### Ambiguous locations

If a location name could belong to multiple timezones (e.g. "Springfield", "Victoria"),
list all possibilities with their IDs and ask the developer to confirm which one applies.

---

## Step 2 — Look up timezone IDs (non-address queries)

Read `references/timezone-index.md` for the Windows ↔ IANA mapping table.

Always provide **both** IDs:
- Windows ID → `TimeZoneInfo.FindSystemTimeZoneById()` (Windows only)
- IANA ID → NodaTime or `TZConvert.GetTimeZoneInfo()` (cross-platform)

---

## Step 3 — Generate code

Read `references/code-patterns.md` for the canonical patterns:

| Scenario | Pattern |
|---|---|
| Simple conversion, Windows-only | Pattern 1 (TimeZoneInfo) |
| Cross-platform / Linux / Docker | Pattern 2 (TimeZoneConverter) — default |
| Complex scheduling, DST arithmetic | Pattern 3 (NodaTime) |
| API response / DB storage | Pattern 4 (DateTimeOffset) |
| ASP.NET Core service | Pattern 5 |
| Recurring jobs (Hangfire/Quartz) | Pattern 6 |
| DST edge cases | Pattern 7 |

Always include the NuGet package reference when recommending a third-party library.

---

## Step 4 — Warn about common pitfalls

Proactively mention when relevant:
- ⚠️ `TimeZoneInfo.FindSystemTimeZoneById()` with IANA IDs **fails on Windows** (and vice versa on Linux) — use `TZConvert.GetTimeZoneInfo()` for cross-platform
- ⚠️ Never store `DateTime.Now` in a database — always `DateTime.UtcNow`
- ⚠️ `DateTimeKind.Unspecified` is a silent bug — always declare intent
- ⚠️ During DST transitions, 1 hour is skipped or repeated — use NodaTime for strict handling
- ⚠️ Azure App Service on Linux uses IANA IDs; on Windows uses Windows IDs

---

## Response Format

For address inputs: location blocks + combined snippet (as above).

For all other queries:
1. Timezone ID block
2. Recommended approach (1 sentence)
3. C# code snippet (copy-paste ready, with using statements)
4. NuGet package if needed
5. One pitfall warning if applicable

Keep responses concise and code-first. Developers want to copy-paste and move on.

---

## Reference Files

- `references/timezone-index.md` — Full Windows↔IANA mapping table for all major zones
- `references/code-patterns.md` — All 7 C# patterns (copy-paste ready)
