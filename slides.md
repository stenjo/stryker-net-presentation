---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Using Stryker mutator
info: |
  ## Are 100% test coverage good enough?
  How to make your unit or component tests bulletproof!

# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
lineNumbers: true
---

# Using Stryker Mutator

## Make your tests bullet proof!

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
layout: default
transition: slide-up
---

# Sten Johnsen

A tech geek spending his work and spare time figuring out stuff involving electronics and software


- **Bouvet** - Since 2008, currently team-lead and (full-stack) developer
![Sten](/images/Sten_Johnsen_6879.jpeg) {width=200px margin=30px align=right}

- **Experience** - Graduated 1991 - B.Eng Microelectronic computer systems. Programming since my first real job - never looked back.
  
- **Roles** - Programmer, project manager, program manager, department head, entrepreneur, agile coach and relationship counsellor

- **Trainer** - DevOps certification courses, Agile, Scrum

- **Busy with** - Quality of software and creating high performing teams

<!--
15+ years in Bouvet, 
Full stack developer, 
teamlead and 
DevOps trainer
-->


---
layout: image-right
image: images/LocationService.png
backgroundSize: contain
---

# Consider some example code (LocationService.cs)
<v-click>

A backend service that handles Locations

</v-click>

<v-clicks>

- implemented in C# .NET<br/>
- 92% test coverage<br/>

</v-clicks>

<v-after>

![test coverage](/images/LocationService-coverage.png)

</v-after>

<!--
[click] Consider this backend endpoint handling Heats
[click] Is entirely coded in C#

[click] 100% test coverage. Not only on 

- lines, but also 
- statements, 
- functions and
- branches
-->

---
layout: image-right
image: images/LocationService.png
---

# LocationService

Lets consider the complete LocationService/LocationService.cs
having automated xUnit tests
in `LocationServiceShould.cs` test suite.

<v-clicks>

- 1040 lines of code<br/>
- 77 tests<br/>
- Provides 92% test coverage<br/>
![LocationService test log](/images/LocationService-test-log.png)

</v-clicks>

<v-click>

How do we know these tests are good?<br/>
(And by good we mean: able to detect bugs we introduce)

</v-click>

<!--
The LocationService provides a set of services to return heats by filters and other related data connected to the Heats

[click] The LocationService service has automated tests implemented in LocationServiceShould.cs - a file with more than 580 lines of code.

[click] contains 77 tests altogether

[click] This test suite provides 92% test coverage.

[click] But are we sure the tests are reliable and will let us know if we later on introduce bugs in the code?
-->

---

# LocationService.cs

Lets test our tests to see how waterthight our testing is by considering a few lines of code with 100% test coverage in the LocationService module:<br/>
> How many bugs can we introduce and still have all tests passing?

<v-click>

````md magic-move {lines: true}
```cs {*|13}{lines:true,startLine:29}
// code
  if (context != null && search.ProjectIds.Count > 0 && !search.allProjects)
  {
      var locationIds = context.Pipes
          .Where(p => search.ProjectIds.Contains(p.ProjectId) && p.CurrLocation != null)
          .Select(p => p.CurrLocation != null ? p.CurrLocation.Id : Guid.Empty)
          .Distinct()
          .ToList();
      query = query.Where(x => locationIds.Contains(x.Id));
  }
  if (!string.IsNullOrEmpty(search.UpdatedDatetime) && DateTime.TryParse(search.UpdatedDatetime, out var updatedSince))
  {
      query = query.Where(q => q.UpdatedDatetime != null && q.UpdatedDatetime >= updatedSince);
  }

```

<Arrow x1="10" y1="20" x2="100" y2="200" />

```cs {13}
// bug 1
  if (context != null && search.ProjectIds.Count > 0 && !search.allProjects)
  {
      var locationIds = context.Pipes
          .Where(p => search.ProjectIds.Contains(p.ProjectId) && p.CurrLocation != null)
          .Select(p => p.CurrLocation != null ? p.CurrLocation.Id : Guid.Empty)
          .Distinct()
          .ToList();
      query = query.Where(x => locationIds.Contains(x.Id));
  }
  if (!string.IsNullOrEmpty(search.UpdatedDatetime) && DateTime.TryParse(search.UpdatedDatetime, out var updatedSince))
  {
      query = query.Where(q => q.UpdatedDatetime != null || q.UpdatedDatetime >= updatedSince);
  }
```

```cs {13}
// bug 2
  if (context != null && search.ProjectIds.Count > 0 && !search.allProjects)
  {
      var locationIds = context.Pipes
          .Where(p => search.ProjectIds.Contains(p.ProjectId) && p.CurrLocation != null)
          .Select(p => p.CurrLocation != null ? p.CurrLocation.Id : Guid.Empty)
          .Distinct()
          .ToList();
      query = query.Where(x => locationIds.Contains(x.Id));
  }
  if (!string.IsNullOrEmpty(search.UpdatedDatetime) && DateTime.TryParse(search.UpdatedDatetime, out var updatedSince))
  {
      query = query.Where(q => q.UpdatedDatetime == null && q.UpdatedDatetime >= updatedSince);
  }
```

```cs {13}
// bug 3
  if (context != null && search.ProjectIds.Count > 0 && !search.allProjects)
  {
      var locationIds = context.Pipes
          .Where(p => search.ProjectIds.Contains(p.ProjectId) && p.CurrLocation != null)
          .Select(p => p.CurrLocation != null ? p.CurrLocation.Id : Guid.Empty)
          .Distinct()
          .ToList();
      query = query.Where(x => locationIds.Contains(x.Id));
  }
  if (!string.IsNullOrEmpty(search.UpdatedDatetime) && DateTime.TryParse(search.UpdatedDatetime, out var updatedSince))
  {
      query = query.Where(q => q.UpdatedDatetime != null && q.UpdatedDatetime > updatedSince);
  }
```

```cs {13}
// bug 4
  if (context != null && search.ProjectIds.Count > 0 && !search.allProjects)
  {
      var locationIds = context.Pipes
          .Where(p => search.ProjectIds.Contains(p.ProjectId) && p.CurrLocation != null)
          .Select(p => p.CurrLocation != null ? p.CurrLocation.Id : Guid.Empty)
          .Distinct()
          .ToList();
      query = query.Where(x => locationIds.Contains(x.Id));
  }
  if (!string.IsNullOrEmpty(search.UpdatedDatetime) && DateTime.TryParse(search.UpdatedDatetime, out var updatedSince))
  {
      query = query.Where(q => q.UpdatedDatetime != null && q.UpdatedDatetime < updatedSince);
  }
```

````

</v-click>

<v-click>

- 4 possible bugs in 1 line of code!

</v-click>

<v-click>

> ## Full test coverage does **not** ensure high code quality

</v-click>

<!--
Let's analyze a small part of the code of the module production code to see if we can introduce bugs that our tests does not detect.

[click] This is a small fraction of the production code, in this particular file

[click] Lets consider only 6 lines of the code and start introduce changes.

[click] - The whole if condition might be inadvertently evaluate to false always

[click] - The first half of the condition might be false

[click] - Behaviour might change to AND from OR

[click] - The last half might be false

[click] - The whole block might be removed

[click] - The second if condition might be set to true

[click] - or to false

[click] - or the whole block might be removed.

[click] None of these changes are caught by any of the tests covering the code.

[click] As a matter of fact, we are able to find 8 possible bugs in these few lines of code

[click] So obviously, even with 100% test coverage, we do not have the absolute top code quality or robustness against future changes

-->

---

# Test quality

In the previous example, there were also 8 bugs that was detected by a test:<br/>

<br/>

$$
\begin{aligned}
\frac{8 \cdot detected}{8 \cdot detected + 4 \cdot undetected} &= \frac{8}{12} &= 0,67 &= 67\%
\end{aligned}
$$

<br/>

<v-click>
A few takeaways:
</v-click>

<v-clicks>

- Although tests are not useless,
  
- Test coverage alone is not good enough

- Test quality can be better

- BUT: -> No test coverage = no tests at all
  
</v-clicks>

<!--
Test quality can be quantified as a percentage of changes found upon changes tested

Turns out that there are 8 changes that was detected by our tests - that we did not cover now.

[click] What we have learned:

[click] - although test coverage is good, coverage alone does not guarantee that all future or present bugs will be detected by our tests.

[click] [click] - There is always room for better tests

[click] - But let's not forget that without test coverage, we have no quality assurance at all, and we don't know what our code works for all relevant situations.
-->

---
layout: image-right
image: images/test-code.png
---

# But Why?

<v-click>
Doing this exercise enables us to:
</v-click>

<v-clicks>

- Find functionality we have not tested
- Detect useless code
- Get  hints to write better tests
- Make our code more robust

</v-clicks>

<br/>

<v-click>

> In this particular case I can add tests to verify edge cases the code is meant to catch, or I could delete some of the code if it turns out it's useless.<br/>
</v-click>

<v-click>
BTW: The overall score for the LocationService module tests is 67%. <br/><br/>
How do I know?
</v-click>

<!--
So, why should we do this exercise?
[click]

[click] We can detect functionality we have not yet good test for. 
(typically a symptom of writing tests when the production code is already done)

[click] There might be parts of the code we really don't need if everything works in spite of inserted changes.

[click] Point to areas where we might improve the tests

[click] Increase the robustness of the code by improving tests so that we feel safer when changing it later
-->

---
layout: image-left
image: images/stryker-man.png
---

# Stryker-mutator

Stryker is an automated way of testing your tests through altering your code by inserting bugs (mutants) and then running your tests to verify that the tests are catching the bugs or not.

<v-clicks>

- open source
- no licenses
- does not leak your code externally
- available for:
  JS/TS, C# and Scala
- produces kill-score for your tests
- all in a handy report

</v-clicks>

<v-after>

![Stryker results](/images/stryker-scores.png)

</v-after>
---
layout: default
---

# How Stryker Works
  <img src="/images/monkeyTest.avif" alt="Monkey Test" style="float: right; height: 250px; margin-left: 20px;"/>

<v-clicks>

- Filters the files to be mutated
  
- Determines what mutations to insert
- Performs a dry-run of all tests to verify all is passing
- Makes a copy of your code, applies one mutation and runs all relevant tests
- Counts the status of the tests for each mutation
- Creates a report based on all mutations

</v-clicks>

---
layout: two-cols-header
class: gap-x-12

---

# Practical use in other projects

Apart from running Stryker from command line, there might also be a benefit from running stryker in a github action. <br/>

::left::

## Nightly

Implemented nightly Stryker test of all code-base,<br/>
storing the report available to the team.

<v-clicks>

- Track total score over time
- Use changes in total score as input to<br/>
  retrospectives

<div class="flex gap-12">
  <div class="w-9/10">

```yaml
name: Stryker mutation tests frontend

on:
  workflow_dispatch:
  schedule:
    - cron: "30 0 * * 1,2,3,4,5"
      
jobs:
...
```
  </div>
</div>

</v-clicks>

::right::

## Pull Requests

Run Stryker on changed code as part of the validation<br/>
checks in pull-requests.

<v-clicks>

- Make it quick for immediate feedback to developers
- Discuss within the team if there should be a minimum score

```yaml
name: Stryker frontend PR diff

on:
  workflow_dispatch:
  push:
    branches:
      -'*'
      -'!main'  
  pull_request:
    paths:
      - 'frontend/src/**/*.tsx'
      - 'frontend/src/**/*.ts'
      
jobs:
...
```
</v-clicks>

---

# Caveats

Stryker, in its nature, runs your test suites for every mutation it decides is relevant. This means that for large codebases a full report takes a long time to complete.

Stryker has ways to reduce time spent:
<v-clicks>

- Incremental mode: Only run mutations changed
- Pre-check code for compilation or build errors
- Option to ignore static mutations
- Ignores mutations in uncovered code
- Run stryker on part of the code only (lines changed)

</v-clicks>
---
layout: two-cols
---

# Our Experience
### Report on pull request
![GitReport](/images/Report_from_git.png) {width=430px margin=30px align=left style="filter: brightness(150%);"}

#### Get feedback on test quality where you are working 
::right::

<br>

<v-clicks>

## What do we do with this information?

<v-click>

- **Drop everything to improve score?** No
</v-click>
<v-click>

- **Act on feedback from new tests**: How good are the new tests?
</v-click>
<v-click>

- **Get information**: How good are the test testing things touched by your new fancy experimental feature?
</v-click>
<v-click>

- **Challenges Faced**: As with any technology added to a project it must be maintained
</v-click>
<v-click>

- **What we get**: A well of information about our tests from little effort (once stryker is set up)
</v-click>

</v-clicks>

---
layout: default
---

<iframe src="mutation-report.html" style="zoom: 0.5; width: 125%; height: 125%; border: none;"></iframe>

---
layout: image-right
image: images/judge.jpg
---
# How to get high stryker score?

<v-clicks>

- Adopt a culture of Quality First
- Arrange for fail-fast and fast feedback
- Focus on Stryker score in peer-reviews
- Do TDD!

</v-clicks>

<!--
[click] A culture of Quality First means we start off with quality. Some times known as shift-left testing.

[click] Make sure we fail fast when we fail! The closer in time from creating the bug to actually know about it, the faster and easier (and cheaper) we can fix it.

[click] Stryker Score is a feedback on your tests. Use it as a source of discussion or in retrospectives.

[click] Test Driven Development is a well known and efficient way of getting to Quality First.
-->

---
layout: image-right
image: images/tdd-img.png
backgroundSize: 30em
---

# TDD

Some other benefits from TDD:
<v-clicks>

- Trustworthy code
- Better architecture
- Tests are the documentation
- Easy when debugging<br/>
and as an extra benefit:
- Guarantees test coverage and high stryker score

</v-clicks>

<v-after>

```
|------------------------|---------|----------|---------|---------|-------------------|
|File                    | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s |
|------------------------|---------|----------|---------|---------|-------------------|
|All files               |     100 |      100 |     100 |     100 |                   |
| ChangeFailureRate.ts   |     100 |      100 |     100 |     100 |                   |
| CommitsAdapter.ts      |     100 |      100 |     100 |     100 |                   |
| DeployFrequency.ts     |     100 |      100 |     100 |     100 |                   |
| IssuesAdapter.ts       |     100 |      100 |     100 |     100 |                   |
| LeadTime.ts            |     100 |      100 |     100 |     100 |                   |
| MeanTimeToRestore.ts   |     100 |      100 |     100 |     100 |                   |
| PullRequestsAdapter.ts |     100 |      100 |     100 |     100 |                   |
| ReleaseAdapter.ts      |     100 |      100 |     100 |     100 |                   |
| index.ts               |     100 |      100 |     100 |     100 |                   |
|------------------------|---------|----------|---------|---------|-------------------|
Test Suites: 13 passed, 13 total
Tests:       93 passed, 93 total
Snapshots:   0 total
Time:        6.607 s
Ran all test suites.
```

</v-after>

<!--
And by the way - TDD has other benefits too:

[click] - Reliable and trustworthy code. We can feel safe that a change done does not break functionality somewhere else without the tests letting us know.

[click] - Architecture of the application is automatically better structured and always a good fit for testing

[click] - Tests tells us how the production code is intended to work, and therefore is good documentation.´Production code alone is never documentation of anything else than how the code works now. 

[click] - Good tests are very easy to use when debugging and can be targeted to very narrow areas.
-->

---
layout: full

---

# References

- Stryker Mutator (web): `https://stryker-mutator.io/`
- Test-Driven Development: `https://docs.google.com/presentation/d/1L5yPzxBlDJ5r56OHpUv1BLO4pkxFYT9V1yLUgosThq4/edit?usp=sharing`
  
![Book](/images/book-clipart-lg.png) {width=300px margin=30px}



---
layout: fact

---

# Thank you!




## ![Presentation](/images/stryker-pres-qr.png) {width=200px margin=30px align=right}
