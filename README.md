# Live Poll — classroom polling you own

A single HTML page. Students scan a QR code on your slide, tap an answer on
their phone, and the bars on the projector move as the votes land. No student
accounts, no per-seat license, no data leaving your own Firebase project.

Based on [XuanyaoLiuLab/livepoll](https://github.com/XuanyaoLiuLab/livepoll),
extended with slides, revisits with before/after, hidden results, student
questions and plain-text import.

Three files:

| file | what it is |
|---|---|
| `index.html` | the whole app — teacher console, student view, live results |
| `firestore.rules` | who is allowed to write what (paste into Firebase) |
| `README.md` | this file |

---

## One-time setup (about 10 minutes)

### 1. Create the Firebase project (free)

1. Go to <https://console.firebase.google.com> and **Add project**. Name it
   anything (`uchicago-polls`). Turn Google Analytics **off** — you don't need it.
   No credit card, no billing: the free Spark plan covers a classroom many
   times over.
2. In the left sidebar: **Build → Firestore Database → Create database**.
   Choose **Start in production mode** (the rules in step 3 replace the
   defaults), and pick a region near you — `nam5 (us-central)` is fine.
3. **Build → Authentication → Get started → Anonymous → Enable → Save.**
   This is what lets a student vote without logging in: their browser silently
   gets a throwaway identity, which is how the app stops one phone from
   voting twenty times.

### 2. Paste your config into `index.html`

In the Firebase console: **⚙ Project settings → Your apps → Web (`</>`)**.
Register an app (any nickname, skip Firebase Hosting). It shows you a
`firebaseConfig` object. Copy it and replace the `PASTE_ME` block near the
top of `index.html`.

Those values are **not secrets** — they ship in every Firebase web app and
are meant to be public. Access is controlled by the rules, not by hiding them.

### 3. Publish the security rules

In the console: **Firestore Database → Rules** tab. Delete what's there,
paste the entire contents of `firestore.rules`, click **Publish**.

These rules enforce:

- a student can only write **their own** vote document, never anyone else's;
- votes are only accepted while **you** have voting open, on a question
  that's showing (on its own or on the current slide);
- students can send questions only while you have them turned on, and each
  student can read only their own;
- only the person who created a poll can edit its question or clear its votes;
- nobody can delete someone else's poll.

### 4. Put it on GitHub Pages

```bash
# in a new folder
git init
git add index.html firestore.rules README.md
git commit -m "Live poll app"
git branch -M main
git remote add origin git@github.com:goldenflaw/poll.git
git push -u origin main
```

Then on GitHub: **Settings → Pages → Source: Deploy from a branch →
`main` / `root` → Save.** A minute later the site is live at

```
https://goldenflaw.github.io/poll/
```

In Firebase, add that site under **Authentication → Settings → Authorized
domains** (`goldenflaw.github.io`) so phones can sign in from it.

The repo must be **public** for free GitHub Pages (that's fine — there's no
secret in it, and student votes live in Firestore, not in the repo).

---

## Using it in class

**One code for the whole lecture.** Students scan once, at the start, and
their page stays live all hour — when you push the next question, it appears
on their phone by itself. They never rescan.

### Before class

1. Open the site on the machine you'll teach from. Click **Start a session**.
2. **Bookmark this page.** The URL ends in `#t/ABCD` — that's your console.
   Set it up on the machine you'll actually present from: editing rights are
   tied to the browser that created the session.
3. Click **+ Add question** as many times as you need. Prepare the whole
   lecture's questions in advance; they sit there until you show them.

Reuse the same bookmark next time and Clear votes, or start a new session for
a new code.

### During class

1. Put the QR code up while students settle in — tell them once to leave the
   tab open. Click **Present** for the projector view; a small QR stays in the
   corner the whole time so anyone who drops out can rejoin silently.
2. Hit **Show** on a question (or <kbd>→</kbd>) to push it to their phones.
   Voting opens automatically.
3. Results stay hidden, so nobody is swayed by the room: the projector and
   phones show only how many have answered. **Close voting** (or
   <kbd>space</kbd>) when you're done. A first round stays hidden after it
   closes too, so it can't influence a later re-test. **Show results** (or
   <kbd>R</kbd>) reveals them whenever you choose, on the projector and phones.
4. <kbd>→</kbd> and <kbd>←</kbd> move between questions, <kbd>space</kbd>
   opens/closes voting, <kbd>R</kbd> shows/hides results, <kbd>P</kbd> re-polls, <kbd>esc</kbd> leaves
   present mode. **Hide** returns everyone to a neutral waiting screen between
   topics.

Going back to an earlier question keeps its votes — nothing is lost by moving
around. Students can change their answer while voting is open, which is what
you want for think-pair-share: poll, discuss, re-poll on the same question.

### Import a whole lecture's polls at once

**Import…** takes plain text and adds everything after your current questions:

```
# Warm-up: true or false?            ← a slide; the questions below go on it
Q: Every mutation in a Mendelian disease gene causes the disease.
- True
- False
---                                  ← ends the slide
Q: A question on its own
- Option A
- Option B
# Revisit: has anything changed?
Q: Every mutation in a Mendelian disease gene causes the disease.
                                     ← no options: brings back the earlier question
// notes like an answer key are ignored
```

The box checks as you type and names any line it can't read. Keep decks
with answer keys out of this repository, since the site is public.

### Slides: several questions at once

**+ Add slide** groups questions so students answer them all on one screen,
and the projector shows all their results side by side. A slide sits in the
lecture order like a question: <kbd>→</kbd> steps onto it, **Show** pushes it.
Use ↑ ↓ to put it where it belongs.

- **Revisit earlier polls.** Make a slide that ticks questions you already
  asked, e.g. "Q1, Q3 and Q4 again" at the end of class. The first time it
  comes up it starts a second round by itself (see Re-poll, below): the first
  answers are kept for comparison and phones start blank.
- **A short survey.** Add questions with *Only show this inside a slide*
  ticked, then put them on a slide. They don't come up on their own when you
  step through.

Up to 12 questions per slide. Editing a slide while it's showing updates
students' phones right away.

### Student questions

Click **Let students ask** in the *Student questions* panel and a box
appears under the poll on every phone. Students type a question and send it.
It's anonymous: you see the text and the time, never who asked. Each student
sees only their own questions, never classmates'.

Questions land in the panel as they come in, oldest unanswered first.
**Show on screen** puts one above the poll results on the projector, in large
type. **Done** (or **Answered**) marks it answered and takes it off the screen,
and the student's phone shows it as answered. In present mode, a small
"N student questions" button in the corner tells you something is waiting.
Clicking it takes you back to the console.

Open questions stream right to left along the bottom of the Present view,
like a news ticker (**Stop scrolling on screen** turns that off). New ones join at the end, and
marking one answered or deleting it takes it off at once. Since questions are
anonymous, keep an eye on the panel while it's on.

**Stop taking questions** removes the box from phones. **Clear all** empties
the list for next time. A phone can send at most one question every 8 seconds,
and each question is limited to 500 characters.

### Re-poll: before and after

A revisit slide starts the second round automatically. Anywhere else,
**Re-poll** (next to Close voting, or <kbd>P</kbd> while presenting) does it by
hand: it saves the current result as the first round, clears the votes and
reopens voting on whatever is showing. The second
round is hidden while people vote too. When you close voting, both rounds
appear together: each option gets a pale *before* bar and a solid *after*
bar, with both percentages ("36% → 64%"), on the projector and on phones. It
works on a single question or a whole slide. **Clear votes** discards both
rounds.

---

## Good to know

**Cost.** Firestore's free tier gives 50,000 document reads and 20,000 writes
a day. A 40-student poll with live results costs roughly 40 writes and a few
thousand reads. You would need dozens of large lectures per day to approach
the limit.

**Staying connected.** The student page is a live subscription, not a loaded
snapshot. It survives the phone locking, the browser backgrounding, and Wi-Fi
dropping — the client reconnects and catches up on its own. It does not
survive a student closing the tab, which is what the corner QR code is for.

**Anonymity.** You see counts, never names. The anonymous Firebase uid is
opaque and isn't shown anywhere in the interface. If a student clears their
browser data, they can vote again — this is a temperature check, not an exam.

**Who can open your console.** Anyone who types your `#t/ABCD` URL sees the
console layout, but the security rules won't let them change anything: only
the browser that created the poll can edit or clear it. Just don't project the
console URL — project the QR code or use Present mode.

**A student on a laptop and a phone** counts as two votes. Unavoidable
without logins, and not worth fixing for classroom use.

**If something looks broken**, the two usual causes are: Anonymous sign-in
wasn't enabled in Authentication, or the rules weren't published. The page
tells you which when a call fails.

**Updating from a version without slides or student questions.** Re-publish
`firestore.rules` (step 3). Until you do, single questions keep working, but
slides won't save, students get no question box, and the console shows a note
saying so. Existing sessions and their votes carry over unchanged.

**Reusing a question next quarter.** Open the old `#t/ABCD` bookmark and hit
Clear votes — the question is still there.
