# Field Register: setup and operation

Version 8.0.0

A supervised research log with a write-once record. Students keep an
academic profile, capture timestamped observations with text, location,
photographs, and voice notes, and record calibration curves whose
statistics are computed from the raw data rather than typed in. Each
student sees only their own register. The supervisor sees every register,
verifies profiles, and countersigns entries. The whole register prints as
a cover-paged PDF suitable for a dissertation appendix or an accreditation
file. No billing account is required.

Files in this folder: `index.html` (the whole application),
`manifest.webmanifest` and `sw.js` (installable app behavior),
`firestore.rules` (the access and integrity rules), and the four icon
files.

---

## Step 1. Create the Firebase project

Go to console.firebase.google.com and create a project. Decline Google
Analytics. No payment method is requested and none is required. Use an
account that represents the institution rather than a personal one if you
can, because the record should outlive any individual's account.

## Step 2. Turn on Google sign-in

Open Authentication, then Sign-in method, and enable the Google provider.
Set the public-facing project name to something your students will
recognize. After you have hosted the app, return to Authentication,
Settings, Authorized domains, and confirm your hosting domain is listed.
Sign-in fails silently on an unlisted domain.

## Step 3. Create the database

Open Firestore Database and choose Create database. Select **production
mode**, never test mode. For location choose `asia-south1` (Mumbai). The
location cannot be changed afterwards.

## Step 4. Register the web app and paste the configuration

In Project settings, General, Your apps, add a Web app. Copy the
`firebaseConfig` object it shows. Open `index.html`, find the block marked
STEP 1, and replace the placeholders. Immediately below, at STEP 2,
replace `supervisor@gmail.com` with your own Gmail address in lower case.
List every supervising faculty member if there is more than one. At STEP 4,
set the ownership line printed in the footer of every report. At
STEP 3, immediately below, put your institution's name, its affiliating
university, and your department. These three lines are printed at the head
of the archive cover page.

This configuration is not a secret. It is embedded in every Firebase web
app and is visible to anyone who opens the page. Access is controlled by
the rules in the next step, not by hiding these values.

## Step 5. Install the security rules

Open `firestore.rules`, put the same email addresses into
`supervisorEmails()` at the top, then paste the whole file into the Rules
tab in the Firebase console and publish.

Do not skip this step and do not leave the default test-mode rules in
place. Those rules permit anyone on the internet to read and write your
database. The rules in this file are what make entries immutable and what
prevent one student from reading another's work.

## Step 6. Put the files on the web

The app must be served over HTTPS, or the camera, microphone, location,
and home-screen installation will all refuse to work.

Firebase Hosting: install Node.js, run `npm install -g firebase-tools`,
`firebase login`, and `firebase init hosting` inside this folder. Choose
your project, set the public directory to `.`, answer no to the
single-page rewrite question, and answer no to overwriting `index.html`.
Then run `firebase deploy`.

GitHub Pages: create a repository, upload all files in this folder to its
root, then in Settings, Pages, choose the main branch and the root folder.
Add the resulting domain to Authorized domains in Step 2.

## Step 7. Give it to the students

Send the link. On Android they open it in Chrome and choose Install app or
Add to Home screen. On iPhone the equivalent is Share, then Add to Home
Screen. Ask each student to sign in once before their first session, since
signing in creates the roster row that makes them visible in your Students
tab.

---

## The integrity model

Every entry carries the authenticated user ID of its author. The rules
permit a read only when that ID matches the person asking, or when the
asking account's verified email is in your supervisor list. A student who
rewrites the query in the browser console still receives nothing.

An entry is written once. The rules permit exactly two later changes: the
author may append a correction or void the entry, and the supervisor may
countersign it once. Every other field, including the text, the
timestamps, the attachment list, the sequence number, and the digest, must
arrive unchanged or the write is rejected. Deletion is refused outright,
for everyone.

Each entry stores `serverAt`, a timestamp written by Google's servers
rather than by the phone, and the rules require it to equal the true
request time. A student cannot backdate work by changing the clock on
their device. The device clock is stored beside it and the entry detail
reports any disagreement over ten minutes.

Entries are numbered in sequence per student and each one's SHA-256 digest
is computed over its own content together with the digest of the previous
entry. This forms a chain in which removing or reordering any entry breaks
every link after it. The banner above the timeline recomputes the whole
chain on every load and reports whether it is intact. The supervisor sees
the same check on each student's page.

Corrections require a stated reason. This is deliberate friction, and it
is the same discipline as striking through a value in a paper notebook and
initialling the change.

Where a photograph carries EXIF metadata, the capture time is read and
stored, and a gap of more than a day between capture and recording is
shown in the interface and in the archive. EXIF is often stripped by
messaging apps and by some browsers, so a missing value means unknown, not
suspicious.

## What this does and does not establish

It makes the record attributable, contemporaneous, and impossible to
revise quietly, which is most of what ALCOA+ asks of a laboratory record.

It does not make the data true. An entry containing a fabricated number,
sealed at the moment it was written, is a permanently fixed fabrication.
The defense against that is evidence, not cryptography: require the
instrument's own exported report or a photograph of the readout, and
enter raw values rather than derived ones so the result is computed
rather than typed.

It is also not independent. You control the Firebase project, so an
administrator could in principle alter the database. Genuine tamper
evidence requires an anchor outside the institution's control, such as an
RFC 3161 trusted timestamp or publishing a periodic digest somewhere the
institute cannot rewrite. Neither is implemented here.

This is not a 21 CFR Part 11 or GLP compliant laboratory notebook.
Compliance additionally requires installation and operational
qualification, standard operating procedures, training records, and
signature controls. The software is one component, never the whole.

## The calibration module

Choosing Calibration curve in the composer records the concentration and
response pairs as raw data. The slope, intercept, r squared, residual
standard deviation, limit of detection, limit of quantitation, and the
back-calculated concentration and relative error at every level are
computed from those raw values and are recomputed each time the entry is
opened. No derived number is ever stored as something a student typed.

Ordinary least squares, 1/x weighting, 1/x squared weighting, and forcing
the line through the origin are all available. The limits of detection and
quantitation follow the residual standard deviation and slope approach,
at 3.3 and 10 times the ratio respectively.

The module reports rather than judges. It flags a back-calculated relative
error above fifteen percent, it warns when a high r squared is accompanied
by a large back-calculated error, and it draws the residual plot, which is
the actual test of linearity. The residual sign-change heuristic is weak
at five or six levels and should not be relied on; read the plot.

Verification: the regression, weighting, and detection-limit code was
checked against numpy's `polyfit` and correlation coefficient and against
a closed-form weighted least squares implementation, agreeing to eight
decimal places for ordinary, weighted, and through-origin fits. Repeat
this check with your own reference data before relying on it for
submitted work, and report the agreement if you publish the tool.

## The PDF archive

Archive PDF in the header renders the whole register as a printable
document. It opens with a cover page carrying the institution, the
student's photograph and full enrollment details, the period covered, the
entry and countersignature counts, the profile verification status, and
signature blocks for the student and the guide. The entries follow in
sequence with their timestamps, coordinates, calibration tables and
statistics, embedded photographs, corrections with their reasons, digests,
and countersignatures. The supervisor can generate the same document for
any student from that student's page.

This is the file to place in a shared Google Drive folder, attach to a
dissertation, or produce during an accreditation visit. It is an archive,
not evidence. A PDF on disk can be edited; the authoritative record with
its server timestamps and digest chain remains in the application. Say so
if anyone asks.

Voice notes cannot be reproduced in print and appear in the archive as a
line noting their existence and duration.

## The student profile

On first sign-in a student completes a profile: full name, enrollment
number, program, semester, academic year, guide, project title,
department, an optional photograph, and an optional mobile number. The app
refuses to record an entry until the required fields are filled, because
an unattributed observation is of little use later.

You verify the profile from the Students tab. Verification locks it: after
that point the student cannot change any of those fields and only you can.
Every change, before or after verification, is kept in a dated history
showing who changed what, from which value to which, rather than
overwriting the old value.

The mobile number is optional, is never printed in the archive, and should
be left blank unless you have a reason to hold it. It is personal data and
collecting it makes you responsible for it under the Digital Personal Data
Protection Act. The photograph is optional too and appears only on the
archive cover page.

Every entry stores its own frozen copy of the identifying fields as they
stood at the moment it was written, and that copy is inside the entry's
digest, so it carries the same immutability as the text. This is why an
entry recorded in Semester III still reports Semester III after the
student moves on, and why a project title revised in March does not
silently rewrite the history of work done in January. Where an entry's
frozen context differs from the current profile, the archive prints the
difference rather than hiding it.

One point about privacy that matters if you keep your own work here.
Anyone on the supervisor list can read every entry in the register,
including yours. With a single supervisor address that is not an issue.
If you add a second faculty member so they can oversee their own students,
they will also be able to read your entries. Where that is unwanted, run a
separate Firebase project for your personal register; it costs nothing and
takes about fifteen minutes to set up.

Entries carry a schema number. Version 2 entries hold no identity
snapshot and version 3 entries hold no dissolution data, and each is
hashed in the form that was current when it was written, so older entries
continue to verify after an upgrade. Do not attempt to
add identity fields to old entries; the rules forbid it and the digest
would no longer match.

## The dissolution module

Choosing Dissolution in the composer records a release study as raw
absorbances and apparatus settings. Nothing derived is typed by the
student: the concentration, the corrected cumulative amount, the percent
released, the kinetic constants, the similarity factors and every other
figure are computed from those absorbances each time the entry is opened.

The slope and intercept can be taken directly from a calibration curve
already recorded in the register, which is the preferred route, because
the whole chain from absorbance to percent released is then traceable to
raw data the student recorded earlier. They can also be typed by hand
when the curve was produced outside the app.

Sampling replacement is handled properly. With replacement the cumulative
amount at each point is the concentration times the bath volume plus the
withdrawn volume times the sum of all earlier concentrations. Without
replacement the bath volume is reduced by each withdrawal before the
calculation. Choose the correct option, because the two differ by several
percent by the last time point.

Five release models are fitted: zero order, first order, Higuchi,
Hixson-Crowell, and Korsmeyer-Peppas. Dissolution efficiency and mean
dissolution time are also reported. The models are ranked by r squared
and the closest fit is named, but the app states plainly in the entry and
in the archive that the models are fitted on different transformed scales,
so their r squared values are not strictly comparable and the ranking is
indicative. Report the model you can justify mechanistically.

Korsmeyer-Peppas is fitted only to points at or below 60 percent release,
and is omitted altogether when fewer than three such points exist rather
than being fitted to unsuitable data. The release exponent is reported
with the interpretation boundaries for a cylindrical matrix and an
explicit warning that those boundaries differ for spheres and films.

f1 and f2 are calculated against a reference profile, which can be typed
in or loaded from another dissolution entry in the same register. The app
checks the usual conditions and says when they are not met: more than one
point above 85 percent release in either profile, fewer than three paired
points, excessive variability at a time point, both profiles above 85
percent within 15 minutes, and fewer than twelve units. A Pearson
correlation between the two profiles is also given. An in vitro in vivo
correlation is deliberately not offered, because it would require plasma
concentration data this register does not hold.

Verification: the concentration, sampling correction, all five kinetic
models, f1, f2, the Pearson coefficient, dissolution efficiency and mean
dissolution time were computed for a worked example both by the shipped
JavaScript and by an independent numpy implementation, and every value
agreed. Repeat this check with your own reference data before relying on
the module for submitted work, and report the agreement if you publish.

## Replicates, figures and reports

**Calibration.** The number of replicate readings per concentration is set
with the plus and minus control above the table, from one to eight. When
more than one reading is entered, the line is fitted to every individual
reading rather than to the level means, because that is what gives an
honest residual standard deviation and therefore honest detection and
quantitation limits. The plotted points are level means with one standard
deviation shown as error bars. Expect the detection limit to rise once
replicates are entered: in the worked example a single reading per level
gave an LOD of 0.092 micrograms per millilitre and triplicate readings with
1.5 percent scatter gave 0.24. The second figure is the honest one.

Two figures are now drawn, the calibration curve and the residual plot,
side by side on a wide screen and stacked on a phone.

**Dissolution.** The number of vessels is set the same way, from one to
twelve. Each vessel is carried through its own cumulative sampling
correction and the vessels are averaged afterwards. This is not the same
as averaging the absorbances first and correcting once, which understates
the spread, and it is the reason the standard deviations reported here are
of the percent released rather than of the absorbance. A vessel with a
missing reading at any time point is excluded from the calculation, with a
note saying so, because a cumulative correction cannot be carried across a
gap. The profile is plotted as mean with standard deviation error bars,
and a reference profile is overlaid as a dashed line when one is given.

**Figure export.** Every figure carries JPG, PNG and PDF buttons. The
image is redrawn at three times scale on a white background with print
colors, so it can go straight into a thesis or a manuscript without
rework. On Android this saves to the downloads folder. On iPhone, Safari
may open the file in a new tab instead, from which it can be shared or
saved.

**Reports.** Saving a calibration or dissolution entry produces a PDF
report automatically, unless the checkbox beside the save button is
cleared. The report carries the institution, the student's identifying
details as frozen into that entry, the method, the raw data, every derived
statistic, the figures, the digest and the countersignature if present.
Every page is numbered and carries the ownership line set at STEP 4. The
same report can be regenerated at any time from the Report PDF button in
the entry detail sheet, and the archive PDF now carries the same footer.

## Finding things again

The register is built to be searched rather than scrolled, which matters
once it holds several projects and a few hundred entries.

Every entry can carry a short title, typed in the field above the main
text box. It is optional for routine laboratory entries but it is the
single most useful field for anything you will want to find again, because
it is shown in bold at the head of the entry in the timeline, it appears in
the detail heading and in both PDF reports, and it is the first thing the
search box matches.

The search box matches every word you type, in any order, against the
entry title, the entry text, the experiment name, the place, the tags, the analyte, the
dissolution medium and product, the entry type, the project title frozen
into the entry, and the text and reasons of any corrections. Matches are
highlighted in the results. Pressing the slash key from anywhere jumps the
cursor into the search box.

Below the filters sits a bar of the twenty-four most used tags with their
counts. Selecting several narrows the results to entries carrying all of
them, so "formulation" plus "failed" finds the failed formulation work and
nothing else. This is the main reason to tag entries at the time of
writing rather than intending to organise them later.

The Projects button opens a summary of every experiment or sub-study in
the register, with the date of the first and last entry, the tags in use,
the number of starred entries and the total. Selecting a row filters the
timeline to that experiment. For someone running several projects at once
this is the fastest route back into a thread of work left a month ago.

Two date boxes beside the filters take an exact range. Setting either one
overrides the quick range selector, so a single date in the first box shows
everything from that day onward, and filling both bounds the search to
those days inclusive. A type selector narrows to observations, calibration
curves or dissolution studies alone.

Any entry can be starred from its detail sheet, and the Starred button
filters to those alone. A star is personal organisation rather than part
of the record: it sits outside the digest, it can be added and removed for
the life of the entry, and it does not affect the integrity chain. This is
the intended way to mark project ideas and decisions you will want to find
again, as distinct from routine experimental entries.

Recording an idea in the register rather than in a notebook file has one
specific advantage worth understanding. The entry carries a server-written
timestamp that cannot be altered afterwards, so the register establishes
when an idea was conceived. That is the evidence that matters if a
question of priority ever arises. The cost is that an idea, once saved,
cannot be edited, and its development has to be written as dated
corrections or as later entries. That is a fair trade for a record of
conception, but it means the register suits the statement of an idea
better than the drafting of one.

## Preserving the record after a student leaves

Export full archive, on the student's page in the Students tab, produces a
single ZIP file holding everything. This is the file to keep when a
student completes their degree, and it is the one to reach for if a
question of priority, authorship or patentability ever arises.

It contains `entries.json`, which is every entry exactly as stored,
including the raw calibration and dissolution readings, the identity
details frozen into each entry, both the device and the server timestamps,
the corrections with their stated reasons, and the digest chain. This is
the authoritative file. Beside it sits `entries.csv` for spreadsheet use,
one CSV per calibration and per dissolution study holding the raw
readings and the computed results, the original photographs and voice
notes as ordinary files under `media`, and `register.pdf`, the readable
archive with its cover page.

It also contains `verify.html`. Open that file in any browser, on any
computer, and select `entries.json`. It recomputes the whole digest chain
and reports whether any entry has been altered, removed or reordered. It
carries a copy of the exact digest recipe used when the entries were
written, it needs no network connection, no account and no software beyond
a browser, and it will still work years from now when this application and
its Firebase project are gone. That property is the point of the archive.

Two cautions about what the archive proves. Each entry holds two
timestamps: the device clock, which can be wrong, and the server clock,
which the database rules required to equal the true request time and which
therefore cannot be backdated from a phone. Rely on the server clock. And
the archive demonstrates that nobody has altered the exported file, not
that the live database was never altered by whoever administers it. For a
stronger claim, keep the Firebase project alive rather than relying on the
export alone, and consider obtaining an independent trusted timestamp over
the ZIP at the moment of export.

Export the archive at the end of each student's project, before their
account becomes inactive, and keep it with your project files. Students
can export their own archive from the button in the header, which is worth
encouraging, since a student who holds their own copy has less reason to
want the original altered.

## Staying inside the free allowance

Firestore's no-cost tier provides roughly one gigabyte of stored data with
a daily allowance in the region of fifty thousand reads and twenty
thousand writes. Photographs are compressed in the browser to about 250
kilobytes, so a gigabyte holds several thousand. Voice notes are recorded
at low bitrate and capped at three minutes. Video is not uploaded;
students keep it on their phones and name the file in the entry text.

Because nothing is ever deleted, storage only grows. With eight students
this is irrelevant for years, but note it before scaling to a full cohort.

## If something does not work

Sign-in does nothing, or reports auth/internal-error: this is the
standalone limitation. The popup flow cannot complete inside an installed
progressive web app, because the popup opens in a separate browser window
and the app returns to the foreground without waiting. From version 7.1.0
the app detects standalone mode and uses the redirect flow there instead.
If it still fails, open the same address in Chrome as an ordinary tab and
sign in once; an installed app shares its storage with Chrome for the same
origin, so the installed icon will then open already authenticated.

Sign-in reports auth/unauthorized-domain: the hosting domain is missing
from Authorized domains under Authentication, Settings.

Sign-in fails immediately with no window opening: check that a support
email is selected under Authentication, Sign-in method, Google. Enabling
the provider without one leaves the OAuth client incompletely
configured.

Entries save but never appear: the rules were not published, or the
supervisor email in the rules does not match the account you signed in
with. Check the browser console for `permission-denied`.

A correction is refused: the reason field was left empty, or the entry has
already been voided.

The Students tab is missing: your signed-in email is not in
`SUPERVISOR_EMAILS` in `index.html`, or it differs in case.

Save entry does nothing and sends you to the Profile tab: a required
profile field is empty.

A student reports that the profile fields are greyed out: the profile has
been verified and is now locked, which is intended. Change it for them
from their page in the Students tab.

The integrity banner reports a break: the most likely cause is two devices
writing entries at the same moment and claiming the same sequence number,
which forks the chain. Ask the student whether they were signed in on two
devices, and record what you find. Treat a break as a question to
investigate, never as proof of misconduct.

The Install app option does not appear: Chrome offers a true installation
only once the service worker is controlling the page, which does not
happen until the second load. Open the address, wait a few seconds, then
reload once. Note that Add to Home screen and Install app are different:
the former makes a plain shortcut that opens in a browser tab, the latter
creates the standalone application. If the app is already installed,
Chrome hides the option, so check the app drawer first. From version 7.2.0
the page carries its own Install app button, which appears on Android and
iPhone and explains what to do when the browser has not yet offered
installation.

The PDF button does nothing: the jsPDF library is loaded from a content
delivery network and needs a network connection on first use.
