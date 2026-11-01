# দৈনিক আত্মপর্যালোচনা — Design + Logic

> **এই ডকুমেন্ট কী:** Dashboard → Tools → **"দৈনিক আত্মপর্যালোচনা"** পেজটার সম্পূর্ণ স্পেক —
> স্ক্রিন ডিজাইন, ডেটা মডেল, স্কোরিং, এজ কেস, ফাইল প্ল্যান ও টেস্ট।
>
> **কোথা থেকে লেখা:** তোমার বর্ণনা (নামাজ · কুরআন · পড়াশোনা · আত্মসমালোচনা) +
> রিপোর বিদ্যমান সার্ভিস (`DeenStore`, `PrayTimesEngine`, `AmalRoutineStore`,
> `ReviewScheduler`, `DeenProgress`) + `models/rule.dart`।
>
> **সোর্সে বর্তমান অবস্থা:** `lib/screens/rules/rules_home_screen.dart` এখনো
> `Rules Phase 1 — Coming soon`। অর্থাৎ এই পেজ **এখনো তৈরি হয়নি** — নিচেরটা ইমপ্লিমেন্টেশনের চুক্তি।
> (যদি তোমার ইনস্টল করা বিল্ডে এটা আগে থেকেই থাকে, ফারাকটা দেখিয়ে দিও — আমি কোডের সাথে মিলিয়ে দেব।)

---

## ০. এক নজরে

| বিষয় | সিদ্ধান্ত |
|---|---|
| পেজের ধরন | দৈনিক self-audit — **রেকর্ড, বিচার নয়** (অ্যাপের বিদ্যমান فلسفة) |
| চারটি অংশ | নামাজ · কুরআন · পড়াশোনা · আত্মসমালোচনা |
| ডেটার একক | এক দিন = এক ডকুমেন্ট, key = `YYYY-MM-DD` |
| নামাজ/কুরআন | **নতুন করে সেভ নয়** — `salah_log` / `amal_log` থেকে পড়া হবে (single source of truth) |
| পড়াশোনা/আত্মসমালোচনা | নতুন Hive box `muhasaba`-তে |
| স্কোর | ০–১০০, ওয়েটেড (নামাজ ৪০ · কুরআন ২০ · পড়াশোনা ২০ · পর্যালোচনা ২০) |
| স্ট্রিক | স্কোর ≥ ৬০ হলে ঐ দিন "সম্পন্ন" |
| রিমাইন্ডার | ডিফল্ট রাত ৯টা, নিজে বদলানো যাবে |
| অফলাইন | সম্পূর্ণ লোকাল — ইন্টারনেট লাগে না |

---

## ১. পেজটা কোথায় বসবে

### ১.১ প্রাইমারি এন্ট্রি — Tools hub

`lib/screens/dashboard_screen.dart` → `_ToolsCardState._entries()`-এ নতুন এন্ট্রি:

```dart
HubEntryData(
  icon: Icons.self_improvement_rounded,
  color: c.teal,                 // বা c.glow
  title: '🪞 দৈনিক আত্মপর্যালোচনা',
  subtitle: 'নামাজ · কুরআন · পড়াশোনা · আত্মসমালোচনা',
  badge: '10',
  builder: (_) => const DailyReviewScreen(),
),
```

> এখন ৯টা এন্ট্রি আছে, badge `'10'` স্বাভাবিকভাবে বসবে।

### ১.২ সেকেন্ডারি এন্ট্রি — Deen home

`lib/screens/deen/deen_home_screen.dart`-এ "Rules & Discipline" কার্ডটা এখন
`RulesHomeScreen` (Coming soon) খোলে। সেটার `onTap` বদলে **প্রথমে এই পেজ** খুলবে,
আর Rules-এর বাকি অংশ পরে যোগ হবে।

### ১.৩ ডিপ-লিংক

`AppNav.key` দিয়ে নোটিফিকেশন ট্যাপে সরাসরি এই স্ক্রিনে আসবে (সেকশন ৫.৯)।

---

## ২. বিদ্যমান কোড — কী কী ব্যবহার করা যাবে

| ফাইল | API | এই পেজে কী দেয় |
|---|---|---|
| `services/deen_store.dart` | `DeenStore.dayKey(DateTime)` | দিনের key — `YYYY-MM-DD` |
| | `DeenStore.dayLog(key)` | ৫ ওয়াক্তের `SalahEntry` (mode + ts) |
| | `DeenStore.setSalah(key, prayer, mode)` | ওয়াক্ত টগল/সেট |
| | `SalahMode { jamaat, alone, qada }` | নামাজের ধরন |
| | `DeenStore.quranMinutesToday()` | আজকের কুরআন মিনিট |
| | `DeenStore.addQuranMinutes(delta)` | মিনিট যোগ |
| | `DeenStore.adhkarDoneToday()` | আজকের যিকির |
| `services/pray_times.dart` | `PrayTimesEngine.compute(...)` | ৫ ওয়াক্তের সময় |
| | `DeenStore.offsets` / `customTimes` | ইউজারের টিউনিং |
| `services/amal_routine.dart` | `AmalRoutineStore.todayProgress()` | রুটিনের অগ্রগতি ০..১ |
| | `AmalRoutineStore.addTodayProgress()` | প্রগ্রেস |
| `services/review_scheduler.dart` | `ReviewScheduler.dueToday()` | আজকের recall বাকি |
| `services/deen_progress.dart` | `DeenProgress.summary()` | মডিউলভিত্তিক সারাংশ |
| `models/rule.dart` | `RuleSubCategory`, `RuleStatus`, `RecoveryType` | ভবিষ্যতে Rules মডিউলে মাইগ্রেশনের ম্যাপ |
| `widgets/glass_card.dart` | `GlassCard` | কার্ডের লুক |
| `theme/app_theme.dart` | `AppColors` | রঙ |

**নীতি:** নামাজ ও কুরআনের ডেটা **দুইবার সেভ করব না**। নতুন box-এ শুধু সেটাই থাকবে
যা আর কোথাও নেই (পড়াশোনা + আত্মসমালোচনা + দিনের স্ন্যাপশট)।

---

## ৩. ডিজাইন

### ৩.১ স্ক্রিন স্ট্রাকচার (wireframe)

```
┌──────────────────────────────────────────────┐
│ ←  দৈনিক আত্মপর্যালোচনা        [⋯]  [⚙]      │  AppBar
│    শনিবার, ১ নভেম্বর ২০২৬ · ১৪ রবিউস সানি    │  Hijri সাবটাইটেল (ঐচ্ছিক)
├──────────────────────────────────────────────┤
│  ╭────────────────────────────────────────╮  │
│  │   ৭২ / ১০০          🔥 ৫ দিন            │  │  Hero score card
│  │   ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░                  │  │  progress ring/bar
│  │   "আজ নামাজে ভালো ছিলে"                  │  │  one-line সারাংশ
│  ╰────────────────────────────────────────╯  │
├──────────────────────────────────────────────┤
│  🕌 নামাজ                            ৪০/৪০   │  Section 1
│  ┌────────────────────────────────────────┐  │
│  │ ফজর     ✓ জামাতে      ৫:০২ → ৫:২১      │  │
│  │ যোহর    ✓ একা          ১:১৫            │  │
│  │ আসর     ○ বাকি         ৪:৩০ পর্যন্ত    │  │  (due highlight)
│  │ মাগরিব  —              ৫:৪৮            │  │
│  │ এশা     ✗ কাজা                         │  │
│  └────────────────────────────────────────┘  │
├──────────────────────────────────────────────┤
│  📖 কুরআন                            ১২/২০  │  Section 2
│  ▓▓▓▓▓▓▓▓░░░░░░  ১২ / ২০ মিনিট               │
│  [+৫] [+১০] [+১৫] [নিজে লিখো]  · ৩ আয়াত    │
├──────────────────────────────────────────────┤
│  📚 পড়াশোনা                          ০/২০   │  Section 3
│  ░░░░░░░░░░░░░░  ০ / ৩০ মিনিট                │
│  বিষয়: [ তাফসির ▾ ]   [+১০] [+২০]  [নোট]    │
├──────────────────────────────────────────────┤
│  🪞 আত্মসমালোচনা                     ২০/২০  │  Section 4
│  আজ কী ভালো হয়েছে?                          │
│  ┌────────────────────────────────────────┐  │
│  │ ফজরে সময়মতো উঠেছি                     │  │
│  └────────────────────────────────────────┘  │
│  কোথায় পিছিয়েছি?                            │
│  ┌────────────────────────────────────────┐  │
│  │ বিকেলে ফোনে ১ ঘণ্টা নষ্ট                │  │
│  └────────────────────────────────────────┘  │
│  কাল কী একটাই জিনিস বদলাব?                    │
│  ┌────────────────────────────────────────┐  │
│  │ আসরের পর ফোন দূরে রাখব                  │  │
│  └────────────────────────────────────────┘  │
│  আজকের অনুভূতি:  😞 😐 🙂 😊 🤲   (১–৫)       │
│  ট্যাগ: #ফোন #ঘুম #রাগ #সময়                │
│                                              │
│  [ সেভ করো ]                                 │
└──────────────────────────────────────────────┘
```

### ৩.২ ভিজুয়াল ভাষা

| টোকেন | মান | নোট |
|---|---|---|
| ব্যাকগ্রাউন্ড | `AppColors` থিমের surface | থিম বদলালেও ঠিক থাকবে |
| কার্ড | `GlassCard(borderRadius: 18, accent: c.glow)` | বাকি অ্যাপের সাথে মিল |
| নামাজের রঙ | `c.income` (সবুজ) | সম্পন্ন |
| বাকি/দেরি | `c.mediumPriority` (অ্যাম্বার) | |
| কাজা/মিস | `c.expense` (লাল) | কিন্তু **গিল্টি টোন নয়** — শুধু তথ্য |
| সংখ্যা বড় | `fontFeatures: [FontFeature.tabularFigures()]` | স্কোর লাফাবে না |
| আরবি | `fontFamily: 'Amiri'` | শুধু আরবি টেক্সটে |
| বাংলা | ডিফল্ট (Noto Sans Bengali) | |
| স্পেসিং | ১৬ px পেজ প্যাডিং, সেকশন গ্যাপ ১২ px | |
| টাচ টার্গেট | সর্বনিম্ন ৪৮×৪৮ | accessibility |

### ৩.৩ সেকশনের বিস্তারিত

#### (ক) হিরো স্কোর কার্ড

- বড় সংখ্যা: `score` (০–১০০)
- রিং: `CircularProgressIndicator(value: score/100)` — stroke ৮
- স্ট্রিক: 🔥 + দিন সংখ্যা
- এক লাইন সারাংশ: সবচেয়ে দুর্বল সেকশন থেকে জেনারেট (নিচে ৫.৬)
- **কোনো শাস্তিমূলক ভাষা নয়** — "পিছিয়ে আছো" নয়, "আসর এখনো বাকি"

#### (খ) নামাজ

- ৫টি সারি, প্রতিটিতে: নাম → স্ট্যাটাস চিপ → সময়
- স্ট্যাটাস চিপ: `✓ জামাতে` / `✓ একা` / `✓ কাজা` / `○ বাকি` / `—`
- ট্যাপ করলে bottom sheet: `জামাতে · একা · কাজা · মুছে ফেলো`
- `বাকি` ওয়াক্তের জন্য পরের ওয়াক্তের সময় দেখানো হয় (deadline)
- `○` চিহ্নে হালকা pulse (২ সেকেন্ড, একবার)

#### (গ) কুরআন

- মিনিট প্রগ্রেস বার (টার্গেট ডিফল্ট ২০ মিনিট)
- কুইক বাটন: +৫ / +১০ / +১৫
- "নিজে লিখো" → নাম্বার প্যাড শিট
- আয়াত সংখ্যা (ঐচ্ছিক, ম্যানুয়াল)
- ডেটা `DeenStore.quranMinutesToday()` থেকেই — আলাদা কপি নয়

#### (ঘ) পড়াশোনা

- মিনিট প্রগ্রেস (টার্গেট ডিফল্ট ৩০)
- বিষয় ড্রপডাউন: `তাফসির · হাদিস · ফিকহ · আরবি · দুনিয়াবি পড়া · অন্যান্য`
- কুইক +১০ / +২০
- "নোট" → ইনলাইন টেক্সট (কী পড়লে) — ঐচ্ছিক

#### (ঙ) আত্মসমালোচনা

তিনটি ছোট টেক্সট ফিল্ড (maxLines 3, maxLength ২০০) + ১–৫ মুড + ট্যাগ চিপ।

ট্যাগ সেট (প্রিসেট, নিজে যোগও করা যাবে):
`#ফোন #ঘুম #রাগ #সময় #পড়া #কাজ #পরিবার #স্বাস্থ্য`

সেভ করলে `review.done = true` এবং ২০ পয়েন্ট।

### ৩.৪ স্টেট

| স্টেট | UI |
|---|---|
| **Loading** | স্কেলেটন (৩টা ধূসর কার্ড), স্পিনার নয় |
| **Empty (ভোর ১২টার পর)** | "আজকের হিসাব শুরু হয়নি" + প্রথম ওয়াক্তের সময় |
| **Partial** | যা আছে দেখাবে, বাকিগুলো `○ বাকি` |
| **Complete (score ≥ ৬০)** | হিরো কার্ডে হালকা glow + "আজকের হিসাব পূর্ণ" |
| **Closed day** (গতকাল দেখা) | ব্যানার: "গতকালের রেকর্ড — সম্পাদনা সীমিত" |
| **Error** | "রেকর্ড পড়া যায়নি" + Retry (Hive ব্যর্থ হলে) |

### ৩.৫ মোশন

- সেকশন কার্ড appear: ২৫ ms stagger, `fade + slide-y 8px`
- স্কোর বদলালে: `TweenAnimationBuilder<int>` ৪০০ ms easeOut
- চেক টগল: `AnimatedSwitcher` ২০০ ms
- **কম মোশন**: `MediaQuery.disableAnimations` হলে সব বন্ধ

### ৩.৬ অ্যাক্সেসিবিলিটি

- প্রতিটি সারিতে `Semantics(label: 'ফজর, জামাতে সম্পন্ন, ৫টা ২১')`
- স্কোর রিং-এ `Semantics(value: '৭২ শতাংশ')`
- রঙ ছাড়াও চিহ্ন (✓ ○ ✗) — রঙ-অন্ধদের জন্য
- ফন্ট স্কেল ২.০ পর্যন্ত লেআউট ভাঙবে না (Flexible + ellipsis)

---

## ৪. ডেটা মডেল

### ৪.১ নতুন Hive box

```dart
// lib/main.dart — বাকি box-গুলোর সাথে
await Hive.openBox('muhasaba');
```

- **key**: `DeenStore.dayKey(date)` → `2026-11-01`
- **value**: `Map<String, dynamic>` (নিচের স্কিমা)

### ৪.২ স্কিমা

```jsonc
{
  "v": 1,                       // স্কিমা ভার্সন
  "prayers_note": "ফজরে দেরি",  // ঐচ্ছিক, নামাজের পাশে ছোট নোট
  "quran": {
    "ayahs": 7,                 // ঐচ্ছিক; minutes আসে amal_log থেকে
    "note": "সূরা ইয়াসিন"
  },
  "study": {
    "minutes": 25,
    "topic": "তাফসির",
    "note": "সূরা বাকারার ১–২০"
  },
  "review": {
    "good": "ফজরে সময়মতো উঠেছি",
    "slip": "বিকেলে ফোনে ১ ঘণ্টা নষ্ট",
    "next": "আসরের পর ফোন দূরে রাখব",
    "mood": 4,                  // 1..5
    "tags": ["ফোন", "সময়"],
    "done": true
  },
  "score": 72,
  "updated": "2026-11-01T21:14:03+06:00",
  "edited": false
}
```

> `prayers` এখানে নেই — কারণ সেটা `salah_log`-এ আছে। এখানে শুধু ডেরাইভড `score`
> রাখা হচ্ছে যাতে ইতিহাস দ্রুত পড়া যায় (পরে লেজি রিকম্পিউটও করা যাবে)।

### ৪.৩ কেন আলাদা box, কেন `salah_log`-এ লিখব না

1. `salah_log` বাংলা-কೀ-ভিত্তিক ৫ ওয়াক্তের জন্য অপ্টিমাইজড — ওখানে review/study ঢোকালে দুই সিস্টেম মিশে যাবে।
2. ভবিষ্যতে **Backup/Export**-এ `muhasaba` একটাই ফাইল হিসেবে যাবে।
3. `models/rule.dart`-এর সাথে ১:১ ম্যাপ করা যাবে:

| এই পেজ | `rule.dart` |
|---|---|
| নামাজ | `RuleSubCategory.salah` |
| কুরআন | `RuleSubCategory.quran` |
| পড়াশোনা | `RuleSubCategory.personalGrowth` / `discipline` |
| আত্মসমালোচনা | `RuleCategory.lifeCharacter` + `RuleVerification.selfCheck` |
| সম্পন্ন/বাকি/কাজা | `RuleStatus.completed / pending / missed` |
| "কাল বদলাব" | `RuleRecoveryAction(type: RecoveryType.custom)` |

---

## ৫. লজিক

### ৫.১ দিনের সীমানা (সবচেয়ে গুরুত্বপূর্ণ)

```dart
String todayKey() => DeenStore.dayKey(DateTime.now());
```

- ডিফল্ট: **স্থানীয় রাত ১২:০০** (অ্যাপের বাকি অংশ এভাবেই চলে)।
- সেটিং (ভবিষ্যৎ): `dayStart = 'midnight' | 'maghrib'`। মাগরিব-ভিত্তিক করলে
  `todayKey()` হবে: মাগরিবের আগে → আগের দিনের key। **ডিফল্ট midnight রাখো**, নাহলে
  বাকি সব মডিউলের সাথে অসঙ্গতি হবে।
- অ্যাপ বন্ধ থেকে সকালে খুললে: `didChangeAppLifecycleState(resumed)`-এ key বদলেছে
  কিনা চেক করো, বদলালে স্ক্রিন রিলোড।

### ৫.২ নামাজ সেকশন

```
লুপ: prayers = [fajr, dhuhr, asr, maghrib, isha]
  entry = DeenStore.dayLog(todayKey())[p]
  যদি entry.done:
      পয়েন্ট = mode == jamaat ? 8 : mode == alone ? 7 : 5   // কাজা = ৫
  নাহলে:
      পয়েন্ট = 0
      যদি এখনো ওয়াক্তের শেষ সময় পার না হয় → স্ট্যাটাস "বাকি"
      নাহলে → স্ট্যাটাস "ছুটে গেছে" (তবু কাজা হিসেবে পরে যোগ করা যাবে)
মোট নামাজ পয়েন্ট = Σ (সর্বোচ্চ ৪০)
```

**দেরি শনাক্ত:** `entry.ts` এবং ওয়াক্তের শেষ সময় (`nextPrayerTime`) তুলনা।
দেরি হলে চিপে ছোট `⏱` আইকন — কিন্তু **পয়েন্ট কাটবে না** (রেকর্ড, শাস্তি নয়)।

**ভুল টগল:** same waqt-এ আবার ট্যাপ → bottom sheet থেকে "মুছে ফেলো" →
`DeenStore.clearSalah(key, prayer)`।

### ৫.৩ কুরআন সেকশন

```
minutes = DeenStore.quranMinutesToday()
target  = settings.quranTargetMin   // ডিফল্ট 20
পয়েন্ট = (minutes / target).clamp(0, 1) * 20
```

- আয়াত সংখ্যা ঐচ্ছিক — শুধু দেখানোর জন্য, স্কোরে প্রভাব নেই।
- `+৫` চাপলে `DeenStore.addQuranMinutes(5)` — বাকি মডিউলও আপডেট হবে (single source)।

### ৫.৪ পড়াশোনা সেকশন

```
study = box.get(todayKey())?['study'] ?? {}
target = settings.studyTargetMin   // ডিফল্ট 30
পয়েন্ট = (study.minutes / target).clamp(0, 1) * 20
```

- "পড়াশোনা" = **দুনিয়াবি/দ্বীনি যেকোনো অধ্যয়ন** — টাস্ক মডিউলের সাথে অটো-লিংক করব না
  (টাস্কে "Study" ট্যাগ থাকলে ভবিষ্যতে অটো-সাজেশন দেওয়া যাবে, কিন্তু ডিফল্টে ম্যানুয়াল)।

### ৫.৫ আত্মসমালোচনা সেকশন

```
r = box.get(todayKey())?['review'] ?? {}
পয়েন্ট = 0
যদি r.good non-empty → +7
যদি r.slip non-empty → +7
যদি r.next non-empty → +6
    // একসাথে ২০
যদি শুধু mood সেট → 0 (আলাদা পয়েন্ট নয়)
```

**কেন ৩টি প্রশ্ন:** "কী ভালো হলো / কোথায় পিছিয়েছি / কাল কী বদলাব" — এটাই
মুহাসাবা-র সহজ, অ-শাস্তিমূলক রূপ। প্রতিটা ঐচ্ছিক; খালি রাখলে সেটার পয়েন্ট যায় না-ও
যেতে পারে (নিচের সেটিং)।

### ৫.৬ স্কোরিং

```
score = নামাজ(0..40) + কুরআন(0..20) + পড়াশোনা(0..20) + পর্যালোচনা(0..20)
```

উদাহরণ:

| অংশ | গণনা | পয়েন্ট |
|---|---|---|
| নামাজ | ৪ জামাতে (৩২) + ১ কাজা (৫) | ৩৭ |
| কুরআন | ১২/২০ মিনিট → 0.6 × 20 | ১২ |
| পড়াশোনা | ২৫/৩০ → 0.833 × 20 | ১৬.৬ → ১৭ |
| পর্যালোচনা | ৩টি উত্তর | ২০ |
| **মোট** | | **৮৬** |

**সারাংশ লাইন জেনারেটর** (হিরো কার্ডে):

```
সবচেয়ে কম পয়েন্টের সেকশন বেছে নাও →
  নামাজ   → "আজ নামাজে ক'টা ওয়াক্ত বাকি"
  কুরআন   → "আজ কুরআনে ৫ মিনিট বাকি"
  পড়াশোনা→ "আজ পড়া হয়নি — ১০ মিনিটও চলবে"
  পর্যালোচনা → "আজকের হিসাবটা লিখে ফেলো"
সব ≥ ৮০% হলে → "আজকের হিসাব পূর্ণ — ধারাবাহিক থাকো"
```

### ৫.৭ স্ট্রিক

```
dayComplete(key) = score(key) >= 60
streak = আজ থেকে পিছনে টানা যত dayComplete; আজ incomplete হলে গestern দিন থেকে গোনা
```

- **আজ incomplete হলে স্ট্রিক ভাঙে না** (দিন এখনো বাকি) — শুধু গতকাল incomplete হলে ভাঙে।
- মিস করা দিন (অ্যাপ খোলেনি) = incomplete।
- ৩০ দিনের হিস্ট্রি রাখলে পুরনো ডেটা না থাকলেও সমস্যা নেই — `box.get(key)` null → 0।

### ৫.৮ Recovery

`models/rule.dart`-এর `RecoveryType` ভাবনা এখানে সরলভাবে:

- কাজা নামাজ হলে `mode = qada` → ৫ পয়েন্ট (০ নয়) — উৎসাহ, অপরাধবোধ নয়।
- মিস করা দিনে পরে গিয়ে "কাজা" টগল করা যাবে **৭ দিন পর্যন্ত** (edit window)।
- ৭ দিনের বেশি পুরনো দিন readonly — "রেকর্ড বদলাবে না"।

### ৫.৯ রিমাইন্ডার

- ডিফল্ট **২১:০০** (`RuleReminder`-এর ডিফল্টের সাথে মিল)।
- `flutter_local_notifications`-এ নতুন id রেঞ্জ `300000+` (প্রার্থনার `200000+` আলাদা)।
- টেক্সট: `"আজকের হিসাব — ২ মিনিট"` → ট্যাপে `AppNav.key` দিয়ে স্ক্রিন।
- দিনে একবার; স্কোর ≥ ৬০ হলে আজকের নোটিফিকেশন বাতিল।
- ব্যবহারকারী বন্ধ করতে পারবে (settings টগল)।

### ৫.১০ ডেটা ফ্লো

```
Hive(salah_log) ──┐
Hive(amal_log)  ──┼──► DailyReviewService.build(dayKey) ──► DailySnapshot
Hive(muhasaba)  ──┘             │
                                ├─► score, streak, summaryLine
                                └─► UI
UI টগল ──► DeenStore.setSalah / addQuranMinutes ──► উপরের box ──► rebuild
UI সেভ ──► MuhasabaStore.saveReview(dayKey, review) ──► muhasaba box
```

---

## ৬. সিউডো-কোড

```dart
// lib/services/muhasaba_store.dart
class MuhasabaStore {
  static const _box = 'muhasaba';
  static Box get box => Hive.box(_box);

  static Map<String, dynamic> day(String key) =>
      Map<String, dynamic>.from(box.get(key) as Map? ?? const {});

  static void saveStudy(String key, {int? minutes, String? topic, String? note}) {
    final d = day(key);
    final s = Map<String, dynamic>.from(d['study'] as Map? ?? {});
    if (minutes != null) s['minutes'] = (s['minutes'] as int? ?? 0) + minutes;
    if (topic != null) s['topic'] = topic;
    if (note != null) s['note'] = note;
    d['study'] = s;
    d['v'] = 1;
    d['updated'] = DateTime.now().toIso8601String();
    box.put(key, d);          // একটাই atomic write
  }

  static void saveReview(String key, Map<String, dynamic> review) { /* একই প্যাটার্ন */ }

  static int score(String key) { /* নিচের সার্ভিসে */ }
}

// lib/services/daily_review_service.dart
class DailySnapshot {
  final int prayerPts, quranPts, studyPts, reviewPts;
  final int score;
  final int streak;
  final String line;
  // ...
}

class DailyReviewService {
  static Future<DailySnapshot> build(String key) async {
    final log = DeenStore.dayLog(key);
    int prayer = 0;
    for (final p in DeenStore.prayers) {
      final e = log[p];
      if (e?.done != true) continue;
      prayer += switch (e!.mode) {
        SalahMode.jamaat => 8,
        SalahMode.alone  => 7,
        SalahMode.qada   => 5,
        null             => 0,
      };
    }
    final qmin = key == DeenStore.dayKey(DateTime.now())
        ? DeenStore.quranMinutesToday()
        : _minutesFromAmalLog(key);
    final quran = ((qmin / 20).clamp(0, 1) * 20).round();

    final d = MuhasabaStore.day(key);
    final studyMin = ((d['study'] as Map?)?['minutes'] as int?) ?? 0;
    final study = ((studyMin / 30).clamp(0, 1) * 20).round();

    final r = (d['review'] as Map?) ?? const {};
    int review = 0;
    if ((r['good'] as String? ?? '').trim().isNotEmpty) review += 7;
    if ((r['slip'] as String? ?? '').trim().isNotEmpty) review += 7;
    if ((r['next'] as String? ?? '').trim().isNotEmpty) review += 6;

    final score = prayer + quran + study + review;
    return DailySnapshot(/* ... */);
  }
}
```

---

## ৭. ফাইল প্ল্যান

| কাজ | ফাইল |
|---|---|
| নতুন store | `lib/services/muhasaba_store.dart` |
| স্কোর/স্ট্রিক/সারাংশ | `lib/services/daily_review_service.dart` |
| স্ক্রিন | `lib/screens/deen/daily_review_screen.dart` |
| সাব-উইজেট (স্কোর কার্ড, নামাজ রো, প্রগ্রেস বার, ট্যাগ চিপ) | `lib/screens/deen/daily_review_widgets.dart` |
| Hive box open | `lib/main.dart` — `await Hive.openBox('muhasaba');` |
| Tools এন্ট্রি | `lib/screens/dashboard_screen.dart` → `_entries()` |
| Deen home কার্ড | `lib/screens/deen/deen_home_screen.dart` → Rules কার্ডের onTap |
| রিমাইন্ডার | `lib/services/deen_notifications.dart` — `scheduleDailyReview()` |
| টেস্ট | `test/muhasaba_store_test.dart`, `test/daily_review_score_test.dart` |

**আনুমানিক কাজ:** ১ দিন (স্ক্রিন ~৪৫০ লাইন, সার্ভিস ~২০০, টেস্ট ~১৫০)।

---

## ৮. এজ কেস

| কেস | আচরণ |
|---|---|
| মধ্যরাত পার হওয়া (অ্যাপ খোলা) | `resumed`/টাইমার চেকে key বদলালে স্ক্রিন রিলোড |
| টাইমজোন বদল | `dayKey` সবসময় লোকাল — পুরনো key অটুট |
| ডিভাইস ঘড়ি পিছিয়ে গেল | key আগের দিনে যেতে পারে; `edited` ফ্ল্যাগ দিয়ে চিহ্নিত |
| প্রথমবার খোলা | empty state, স্কোর ০, স্ট্রিক ০ |
| শুধু নামাজ ট্র্যাক, কিছু লেখেনি | score ৪০, স্ট্রিক ভাঙবে (৬০ threshold) |
| কুরআন target ০ সেট | ডিভিশন-বাই-জিরো এড়াতে `max(target, 1)` |
| কাজা যোগ করা | পয়েন্ট ৫, `⏱` নেই |
| পুরনো দিন (৭ দিনের বেশি) | readonly, সম্পাদনা বাটন হাইড |
| Hive box খোলা নেই | `Hive.isBoxOpen` গার্ড; না থাকলে স্ক্রিন error state |
| থিম বদল | `AppColors` রি-রিড, `GlassCard` অটো |
| বড় ফন্ট (২.০×) | সারি `Flexible` + `ellipsis`; নিচে স্ক্রল |
| ব্যাকআপ (ভবিষ্যৎ) | `muhasaba` box JSON-এ যাবে |

---

## ৯. টেস্ট কেস

```
✅ dayKey: 2026-11-01 → '2026-11-01'; 2026-01-05 → '2026-01-05'
✅ নামাজ: 0/5 → 0 পয়েন্ট
✅ নামাজ: 3 জামাতে + 2 বাকি → 24
✅ নামাজ: 5 কাজা → 25
✅ নামাজ: জামাতে(8) > একা(7) > কাজা(5)
✅ কুরআন: 0 মিনিট → 0; 20 মিনিট → 20; 60 মিনিট → 20 (cap)
✅ পড়াশোনা: 15/30 → 10
✅ পর্যালোচনা: দ্বিতীয় ফিল্ড ফাঁকা → 13
✅ মোট: (37+12+17+20) = 86
✅ স্ট্রিক: গতকাল ৭০, আজ ২০ → স্ট্রিক অটুট (আজ এখনো বাকি)
✅ স্ট্রিক: গতকাল ৪০ → স্ট্রিক ০
✅ ৮ দিন আগের key সেভ → readonly, exception নয়
✅ saveStudy দুইবার → minutes যোগ হয় (replace নয়)
✅ স্কোর 0..100 এর বাইরে যাবে না (clamp)
```

---

## ১০. যা ইচ্ছাকৃতভাবে করছি না

1. **ধর্মীয় "গ্রেড"** — "তুমি ৬০% মুসলিম" জাতীয় কিছু নয়। স্কোর শুধু **অ্যাপের ভেতরের
   ধারাবাহিকতা**, ইবাদতের মূল্যায়ন নয়। স্ক্রিনে এটা স্পষ্ট লেখা থাকবে।
2. **অন্যকে দেখানো/শেয়ার** — এটা ব্যক্তিগত; social feature নয়।
3. **AI দিয়ে অটো-জাজ** — AI লিখবে না "তুমি পাপ করেছ"। বড়জোর একটা নিরপেক্ষ সারাংশ।
4. **সালাত-স্কোর দিয়ে নোটিফিকেশন হুমকি** — "তুমি ব্যর্থ" টোন নিষিদ্ধ।
5. **আলাদা দ্বিতীয় review system** — `ReviewScheduler` (Leitner) আলাদাই থাকবে;
   এটা তার সাথে মেশানো হবে না।

---

## ১১. খোলা প্রশ্ন (তোমার সিদ্ধান্ত দরকার)

1. **দিনের শুরু:** রাত ১২টা (ডিফল্ট) নাকি মাগরিব?
2. **নামাজের পয়েন্ট:** জামাতে ৮ / একা ৭ / কাজা ৫ — ঠিক আছে? নাকি সবই সমান?
3. **"পড়াশোনা"** মানে শুধু দ্বীনি পড়া, নাকি যেকোনো পড়া (স্কুল/কলেজ/স্কিল)?
4. **থ্রেশহোল্ড:** ৬০ নাকি ৭০-এ দিন "সম্পন্ন"?
5. **রিমাইন্ডার:** রাত ৯টা ঠিক আছে, নাকি এশার পরে?
6. **ট্যাগ লিস্ট** ঠিক আছে, নাকি তোমার নিজের লিস্ট দেবে?

---

## পরিশিষ্ট — এই ডকুমেন্টে ব্যবহৃত কোড-রেফারেন্স

| দাবি | প্রমাণ |
|---|---|
| Rules স্ক্রিন placeholder | `lib/screens/rules/rules_home_screen.dart:9` — `'Rules Phase 1 — Coming soon'` |
| Tools-এ ৯টা এন্ট্রি | `lib/screens/dashboard_screen.dart:793` — `_entries()` |
| নামাজ মডেল | `lib/services/deen_store.dart:6` — `enum SalahMode { jamaat, alone, qada }` |
| দিনের key | `lib/services/deen_store.dart:155` — `dayKey()` |
| ওয়াক্ত লগ | `lib/services/deen_store.dart:255` — `dayLog()` |
| কুরআন মিনিট | `lib/services/deen_store.dart:501` — `quranMinutesToday()` |
| Rule মডেল | `lib/models/rule.dart:3` — `RuleCategory`, `RuleSubCategory`, `RuleStatus` |
| রিমাইন্ডার ডিফল্ট ২১:০০ | `lib/models/rule.dart:27` — `RuleReminder(hour: 21)` |
| Hive box প্যাটার্ন | `lib/main.dart` — ৩০টা `openBox` |
