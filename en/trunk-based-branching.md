#### Facebook

[InfoQ Lecture from Chuck Rossi](http://www.infoq.com/presentations/Facebook-Release-Process) - Release Engineer at Facebook:
- 50-300 commits every day
- ~30 teams
- Release on every Thuesday
- Have a person who knows the order of what's released - Release Engineer
- They have www.latest.facebook.com - changes that are built everyday and are 'released' to own facebook personnel (Dog Fooding)
- "Report a Bug" functionality is easily accessible while Dog Fooding which gathers all the metrics and possible data for the page
- Preproduction version (www.inyour.facebook.com) where devs test changes together with other changes by others
- Oncall dury - someone whom you contact from dev teams if you find issues
- Branching
 - Shared trunk where everyone commits.
 - Latest branch where only a Gatekeeper can commit. He gathers changes continuously from trunk and commit them into 'latest'. There are 50-300 cherry picks every day.
 - Release branch that gets replaced with new branch on every release.
- Tools:
 - A single IRC for everyone in dev
 - Tools that shows status on what was merged or not
 - A bot that asks everyone who's interested in release to confirm they're fine with their changes to be merged into release branch
 - If tests fail, there is a description of what failed and why so that it's easy to analyze them before release. There is a full regression and "small" main tests (4000 of them).
 - Log Consolidator - a tool that can gather logs from a lot of machines, you can search for lines. If error happens, it's possible to expand the log line and get the stack trace. If you press on a stack trace line, a Git Blame will show the code line. It's also possible to assign users to log lines (errors).
 - Error analysis can show when the error started to appear, what correlated events happened (commits).
 - Risk Ratings - is the change big? The rating gets higher. Are there a lot of disputes in review comments? The rating gets higher.
 - Push Karma - each developer has a rating of how many bugs 
- Feature Gatekeepers - it's possible to assign a condition to the feature when to appear on PROD. The feature might be there for a half a year, but it will be activated only if condition is true. Possible Feature Gatekeepers: Age, Locale, Country, Client IP, OS, etc. Therefore you can test in production.
- Culture: everyone knows that release process is important. If DevOps needs a tool - it's written by devs.
