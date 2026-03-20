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

## How Claude knows the timezone

<svg width="100%" viewBox="0 0 680 480" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- Layer 1: Training data (bottom) -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="40" y="380" width="600" height="60" rx="12" stroke-width="0.5" style="fill:rgb(68, 68, 65);stroke:rgb(180, 178, 169);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="404" text-anchor="middle" dominant-baseline="central" style="fill:rgb(211, 209, 199);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Claude training data</text>
    <text x="340" y="424" text-anchor="middle" dominant-baseline="central" style="fill:rgb(180, 178, 169);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Wikipedia, IANA tz database docs, OpenStreetMap, geographic references, dev forums</text>
  </g>

  <!-- Layer 2: What Claude learned -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="40" y="270" width="186" height="76" rx="10" stroke-width="0.5" style="fill:rgb(60, 52, 137);stroke:rgb(175, 169, 236);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="133" y="296" text-anchor="middle" dominant-baseline="central" style="fill:rgb(206, 203, 246);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Geographic knowledge</text>
    <text x="133" y="316" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Every city, country,</text>
    <text x="133" y="332" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">region mapped to zone</text>
  </g>

  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="247" y="270" width="186" height="76" rx="10" stroke-width="0.5" style="fill:rgb(60, 52, 137);stroke:rgb(175, 169, 236);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="296" text-anchor="middle" dominant-baseline="central" style="fill:rgb(206, 203, 246);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">IANA tz database</text>
    <text x="340" y="316" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">600+ zone IDs,</text>
    <text x="340" y="332" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">history + DST rules</text>
  </g>

  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="454" y="270" width="186" height="76" rx="10" stroke-width="0.5" style="fill:rgb(60, 52, 137);stroke:rgb(175, 169, 236);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="547" y="296" text-anchor="middle" dominant-baseline="central" style="fill:rgb(206, 203, 246);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Windows TZ registry</text>
    <text x="547" y="316" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Windows ID &lt;-&gt; IANA</text>
    <text x="547" y="332" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">full mapping table</text>
  </g>

  <!-- Arrows from training to learned -->
  <line x1="133" y1="380" x2="133" y2="348" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="340" y1="380" x2="340" y2="348" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="547" y1="380" x2="547" y2="348" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- Layer 3: Reasoning at query time -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="40" y="150" width="600" height="76" rx="10" stroke-width="0.5" style="fill:rgb(8, 80, 65);stroke:rgb(93, 202, 165);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="177" text-anchor="middle" dominant-baseline="central" style="fill:rgb(159, 225, 203);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Claude reasoning at query time (no hardcoding needed)</text>
    <text x="340" y="199" text-anchor="middle" dominant-baseline="central" style="fill:rgb(93, 202, 165);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">"42 Galle Road, Colombo" -&gt; country=Sri Lanka -&gt; primary zone -&gt; Asia/Colombo -&gt; UTC+05:30, no DST</text>
    <text x="340" y="215" text-anchor="middle" dominant-baseline="central" style="fill:rgb(93, 202, 165);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">"Denver, CO" -&gt; US Mountain time -&gt; checks DST applicability -&gt; America/Denver</text>
  </g>

  <!-- Arrows from learned to reasoning -->
  <line x1="133" y1="270" x2="250" y2="228" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="340" y1="270" x2="340" y2="228" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="547" y1="270" x2="430" y2="228" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- Layer 4: Skill reference (what skill adds) -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="40" y="50" width="286" height="70" rx="10" stroke-width="0.5" style="fill:rgb(99, 56, 6);stroke:rgb(239, 159, 39);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="183" y="76" text-anchor="middle" dominant-baseline="central" style="fill:rgb(250, 199, 117);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Skill reference index</text>
    <text x="183" y="96" text-anchor="middle" dominant-baseline="central" style="fill:rgb(239, 159, 39);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Curated common zones + C# patterns</text>
    <text x="183" y="112" text-anchor="middle" dominant-baseline="central" style="fill:rgb(239, 159, 39);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">for fast, consistent answers</text>
  </g>

  <!-- Output -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="354" y="50" width="286" height="70" rx="10" stroke-width="0.5" style="fill:rgb(39, 80, 10);stroke:rgb(151, 196, 89);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="497" y="76" text-anchor="middle" dominant-baseline="central" style="fill:rgb(192, 221, 151);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Output to developer</text>
    <text x="497" y="96" text-anchor="middle" dominant-baseline="central" style="fill:rgb(151, 196, 89);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Windows ID + IANA ID + UTC offset</text>
    <text x="497" y="112" text-anchor="middle" dominant-baseline="central" style="fill:rgb(151, 196, 89);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">+ DST flag + C# code snippet</text>
  </g>

  <!-- Arrows reasoning to output and skill -->
  <line x1="280" y1="150" x2="183" y2="122" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="400" y1="150" x2="497" y2="122" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
</svg>

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
