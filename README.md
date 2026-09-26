# Field Register

A timestamped, tamper-evident research log for supervised postgraduate
project work in the pharmaceutical sciences.

Students record observations from a phone as text, voice notes or
photographs, and enter calibration curves and dissolution studies as raw
readings. Every derived quantity is computed by the application from those
raw readings rather than typed in, and each entry is written once and
cannot afterwards be edited or deleted. The supervisor sees every
student's register, verifies profiles, and countersigns entries. Students
cannot see one another's work.

The whole application is a single self-contained HTML file with a Firebase
backend. It installs to an Android or iPhone home screen as a progressive
web app and works offline.

**Live instance:** https://devvaishnav137-sci.github.io/research-register/

---

## Why it exists

Small pharmacy colleges rarely run an electronic laboratory notebook. Free
and capable ones exist, but they assume a research IT group to keep a
server running, which a teaching institution usually does not have. This
project takes the opposite approach: no server to maintain, no licence
fee, no billing account, and a deployment that one member of faculty can
complete in an afternoon.

It is aimed at the postgraduate project cohort rather than at an industrial
laboratory, and it models a supervisor and a batch of students rather than
a principal investigator and salaried staff.

## What it does

**Recording.** Text entries, voice notes recorded in the browser,
photographs compressed on the device, coordinates where the student
permits it, and a server-written timestamp that no device clock can
influence.

**Integrity.** Entries are written once. The only permitted later changes
are an appended correction with a stated reason, a void that leaves the
entry visible and marked, a supervisor countersignature, and a personal
star. The text, timestamps, attachment list and identity snapshot cannot
change, and deletion is refused for everyone. Each entry carries a
SHA-256 digest computed over its own content together with the digest of
the previous entry, so removing or reordering anything breaks the chain,
and the chain is reverified on every load.

**Calibration curves.** Ordinary and weighted least squares, optionally
forced through the origin, fitted to every individual replicate reading
rather than to level means so that the residual standard deviation and the
detection and quantitation limits are honest. Reports the equation, r
squared, s(y/x), LOD, LOQ and the back-calculated error at each level,
draws the curve with standard deviation error bars beside the residual
plot, and warns when a high r squared is contradicted by a large
back-calculated error.

**Dissolution studies.** Per-vessel cumulative release with proper
sampling correction for replaced or unreplaced medium, averaged across
vessels afterwards rather than before. Zero order, first order, Higuchi,
Hixson-Crowell and Korsmeyer-Peppas models, dissolution efficiency, mean
dissolution time, and f1, f2 and Pearson correlation against a reference
profile, with the usual f2 validity conditions checked and reported.

**Output.** Figures export as JPG, PNG or PDF at three times scale on a
white background for direct use in a thesis. Each calibration or
dissolution entry produces a PDF report, and the whole register exports as
a cover-paged PDF archive suitable for a dissertation appendix or an
accreditation file.

**Retrieval.** Titles, multi-word search with highlighting, tag facets,
exact date ranges, entry-type filtering, starring, and a project summary
view.

## What it is not

It makes a record attributable, contemporaneous and impossible to revise
quietly. It does not make the data true: an entry containing a fabricated
number, sealed at the moment it was written, is a permanently fixed
fabrication. The defence against that is evidence, which is why the
application asks for the instrument's own export or a photograph of the
readout and computes results from raw values rather than accepting typed
ones.

It is also not independent. Whoever administers the Firebase project could
in principle alter the database. Genuine tamper evidence would require an
anchor outside the institution's control, such as an RFC 3161 trusted
timestamp, which is not implemented here.

This is not a 21 CFR Part 11 or GLP compliant laboratory notebook.
Compliance additionally requires installation and operational
qualification, standard operating procedures, training records and
signature controls. Do not rely on it alone for a regulatory submission.

## Repository contents

| File | Purpose |
| --- | --- |
| `index.html` | The entire application, including all configuration |
| `firestore.rules` | Database access and immutability rules |
| `manifest.webmanifest`, `sw.js` | Installable app behaviour and offline shell |
| `icon-*.png`, `apple-touch-icon.png` | Launcher icons |
| `SETUP.md` | Full deployment and operation guide |

## Deploying your own instance

Read `SETUP.md` for the detailed version. In outline:

1. Create a Firebase project. Decline Analytics. No payment method is
   required at any point.
2. Enable Google sign-in under Authentication.
3. Create a Firestore database in **production mode**, choosing a region
   near you. Keep the default database, since only that one qualifies for
   the no-cost quota.
4. Register a web app and paste its configuration into `index.html` at
   STEP 1. Set the supervisor email at STEP 2, the institution at STEP 3
   and the report ownership line at STEP 4.
5. Put the same supervisor email into `firestore.rules`, then paste the
   file into the Rules tab in the Firebase console and publish. This step
   is what enforces immutability and per-student isolation, and skipping
   it leaves the database open.
6. Serve the files over HTTPS from GitHub Pages or Firebase Hosting, and
   add the resulting domain to Authorized domains under Authentication.

The Firebase configuration in `index.html` is not a secret. It is embedded
in every Firebase web application and is visible to anyone who opens the
page. Access is controlled by the published rules, not by concealing those
values.

## Running costs

Nothing, at the scale this is designed for. Firestore's no-cost tier
provides roughly one gibibyte of stored data with a daily allowance in the
region of fifty thousand reads and twenty thousand writes. Photographs are
compressed to about 250 kilobytes and voice notes are capped at three
minutes. Video is deliberately not uploaded; students keep it on their
phones and name the file in the entry text. Because the project runs on the
free plan with no billing account attached, exceeding a quota stops the
service until the following day rather than producing a bill.

## Citation

If this tool contributes to work you publish, please cite it. A formal
citation and DOI will be added here on first release.

## Licence and ownership

Field Register is owned by Prof. Devendra Vaishnav, Shree Naranjibhai
Lalbhai Patel College of Pharmacy, affiliated with Gujarat Technological
University. Every report generated by the application carries this
ownership line in its page footer.

An open-source licence has not yet been attached. Until one is added,
all rights are reserved. Adding a permissive licence such as MIT is
recommended before the repository is shared beyond the institution, so
that the tool can be forked and kept alive by others if it is ever no
longer maintained here. Research records depend on the software that holds
them, and abandonment without a licence would leave users stranded.

## Support

This is maintained as an academic side project and is offered without
warranty or any service level. Issues and suggestions are welcome through
the repository issue tracker.
