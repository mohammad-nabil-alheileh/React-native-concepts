# Play Store internal testing

Internal testing is a release track in Google Play Console that lets you distribute your AAB to a small group of testers (up to 100) through the real Play Store infrastructure, before anything is public.

## Release tracks

| Track | Audience | Review by Google? |
| --- | --- | --- |
| Internal testing | Up to 100 testers you list by email | No (available within minutes) |
| Closed testing | Larger invited groups (email lists or Google Groups) | Yes |
| Open testing | Anyone who opts in via the Play listing | Yes |
| Production | Everyone | Yes |

## Steps

1. Create the app in Play Console (name, language, free or paid, declarations).
2. Go to **Testing → Internal testing → Create new release**.
3. Upload your signed AAB. Opt in to Play App Signing if prompted.
4. Add release notes and save, then roll out.
5. Create a tester list: add Gmail addresses (or a Google Group) under the **Testers** tab.
6. Share the opt-in link. Testers open it, accept the invite, and install from the Play Store.
7. Ship fixes: bump `versionCode`, build a new AAB, upload it, and testers get the update automatically.
