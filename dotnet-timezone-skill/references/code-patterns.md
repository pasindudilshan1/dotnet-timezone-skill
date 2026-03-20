# .NET Timezone Code Patterns

## Pattern 1: Basic TimeZoneInfo (Built-in, Windows-only safe)

```csharp
// Convert UTC → target timezone
DateTime utcNow = DateTime.UtcNow;
TimeZoneInfo sriLankaTz = TimeZoneInfo.FindSystemTimeZoneById("Sri Lanka Standard Time");
DateTime localTime = TimeZoneInfo.ConvertTimeFromUtc(utcNow, sriLankaTz);

// Convert local → UTC
DateTime backToUtc = TimeZoneInfo.ConvertTimeToUtc(localTime, sriLankaTz);

// Convert between two non-UTC zones
TimeZoneInfo tokyoTz = TimeZoneInfo.FindSystemTimeZoneById("Tokyo Standard Time");
DateTime tokyoTime = TimeZoneInfo.ConvertTime(localTime, sriLankaTz, tokyoTz);
```

⚠️ **Windows IDs only work on Windows.** On Linux/Docker, use IANA IDs or TimeZoneConverter.

---

## Pattern 2: Cross-Platform with TimeZoneConverter (Recommended)

```xml
<!-- NuGet -->
<PackageReference Include="TimeZoneConverter" Version="6.*" />
```

```csharp
using TimeZoneConverter;

// Works on both Windows AND Linux
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");  // IANA ID
// OR
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Sri Lanka Standard Time"); // Windows ID

DateTime converted = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

---

## Pattern 3: NodaTime (Best for complex scenarios)

```xml
<PackageReference Include="NodaTime" Version="3.*" />
```

```csharp
using NodaTime;

// Current instant in a timezone
DateTimeZone colomboZone = DateTimeZoneProviders.Tzdb["Asia/Colombo"];
Instant now = SystemClock.Instance.GetCurrentInstant();
ZonedDateTime colomboTime = now.InZone(colomboZone);

// Convert between zones
DateTimeZone tokyoZone = DateTimeZoneProviders.Tzdb["Asia/Tokyo"];
ZonedDateTime tokyoTime = colomboTime.WithZone(tokyoZone);

// Parse a local datetime into a zone
LocalDateTime localDt = new LocalDateTime(2024, 6, 15, 14, 30, 0);
ZonedDateTime zoned = colomboZone.AtStrictly(localDt);
Instant utcInstant = zoned.ToInstant();
```

---

## Pattern 4: DateTimeOffset (Modern Best Practice for APIs)

```csharp
// Always store/pass DateTimeOffset, not DateTime
DateTimeOffset utcNow = DateTimeOffset.UtcNow;

TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTimeOffset colomboTime = TimeZoneInfo.ConvertTime(utcNow, tz);

// colomboTime.Offset == +05:30
// Safe to serialize, compare across zones
```

---

## Pattern 5: ASP.NET Core — Storing in DB, Returning in API

```csharp
// ✅ Store as UTC in database
entity.CreatedAtUtc = DateTime.UtcNow;

// ✅ Convert for display based on user's timezone preference
public DateTimeOffset ToUserTime(DateTime utc, string userIanaTimezone)
{
    var tz = TZConvert.GetTimeZoneInfo(userIanaTimezone);
    return TimeZoneInfo.ConvertTimeFromUtc(utc, tz);
}

// ✅ Serialize with offset for APIs
// Use DateTimeOffset — JSON serializer includes the +05:30 automatically
```

---

## Pattern 6: Scheduling / Recurring Jobs (Hangfire / Quartz)

```csharp
// Convert "run at 9am Colombo time" → UTC for scheduler
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("Asia/Colombo");
DateTime scheduledLocal = new DateTime(2024, 12, 1, 9, 0, 0, DateTimeKind.Unspecified);
DateTime scheduledUtc = TimeZoneInfo.ConvertTimeToUtc(scheduledLocal, tz);

// In Hangfire with timezone support:
RecurringJob.AddOrUpdate("morning-job",
    () => DoWork(),
    "0 9 * * *",
    new RecurringJobOptions { TimeZone = tz });
```

---

## Pattern 7: Handling DST Ambiguous/Invalid Times

```csharp
TimeZoneInfo tz = TZConvert.GetTimeZoneInfo("America/New_York");
DateTime ambiguous = new DateTime(2024, 11, 3, 1, 30, 0); // Falls in DST gap

if (tz.IsAmbiguousTime(ambiguous))
{
    // Pick standard time (the later offset)
    var offsets = tz.GetAmbiguousTimeOffsets(ambiguous);
    var standardOffset = offsets.Min(); // Standard is always less
    var dto = new DateTimeOffset(ambiguous, standardOffset);
}

if (tz.IsInvalidTime(ambiguous))
{
    // Time doesn't exist — spring forward gap
    // Add 1 hour to skip past it
    ambiguous = ambiguous.AddHours(1);
}
```

---

## Common Mistakes to Avoid

| ❌ Wrong | ✅ Right |
|---|---|
| `DateTime.Now` in server code | `DateTime.UtcNow` |
| Storing `DateTime.Kind = Local` | Always store UTC |
| Hardcoding `+05:30` offset | Use named timezone ID |
| `FindSystemTimeZoneById("Asia/Colombo")` on Windows | Use `TZConvert.GetTimeZoneInfo("Asia/Colombo")` |
| Comparing `DateTime` across zones | Use `DateTimeOffset` or convert both to UTC first |
| `new DateTime(...)` without specifying Kind | Always specify `DateTimeKind.Utc` or `Unspecified` |

---

## Decision Guide: Which approach to use?

```
Is your app cross-platform (Linux/Docker/Azure Linux)?
  YES → Use TimeZoneConverter or NodaTime (IANA IDs)
  NO (Windows only) → TimeZoneInfo with Windows IDs is fine

Do you need DST-aware arithmetic / scheduling?
  YES → NodaTime (most robust)
  NO → TimeZoneConverter + TimeZoneInfo is sufficient

Are you building an API?
  → Always use DateTimeOffset for serialization
  → Store UTC in database, convert at presentation layer

Do you need all timezone data at compile time?
  → NodaTime bundles TZDB — no OS dependency
```
