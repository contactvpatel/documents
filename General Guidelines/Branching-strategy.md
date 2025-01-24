# Branching Strategy and Release management

1. Importance
2. Influencing factors
3. Branches
4. Workflow

## 1. Importance

Branching strategy is important in software development for several reasons:

1. **Isolation of Changes**: Developers can work in parallel on multiple stories, tasks, or bug fixes without interfering with each other. Developers can work on separate branches without affecting the main codebase, reducing conflicts and streamlining development.
2. **Code Quality**: Branching allows for code reviews and testing to be conducted on isolated changes before they are merged into the main codebase. This helps maintain the overall quality of the code and prevents potentially bugs or incomplete changes from being integrated.
3. **Release Management**: Branches facilitate the management of releases by providing a controlled environment for stabilizing and preparing software versions for deployment. Stable branches can be created for releases while development can continue on separate branches. This also enables the ease of deployment to the different environments.
4. **Story Development**: Branching enables teams to work on new stories independently. This promotes modularity and allows for easier tracking of development.
5. **Experimentation and Prototyping**: Developers can create experimental branches to test out new ideas or prototypes without affecting the main codebase. This allows for innovation and exploration while minimizing risk.
6. **Hotfixes and Maintenance**: Branches provide a mechanism for addressing critical issues or bugs in production code by allowing for the creation of hotfix branches. These branches can be quickly deployed to resolve issues without disrupting ongoing development work.

## 2. Influencing factors

In deciding the following branching strategy, the following factors have been taken into consideration

- Team Size and Structure
- Project Complexity
- Release Frequency
- Risk Tolerance
- Project Lifecycle

Based on the above factors for most of our projects we are selecting the following method as our branching strategy. This method is not the same as git flow or trunk development, it is a strategy that has been tailored for the department.

## 3. Branches

There will be <ins>two permanent</ins> branches and <ins>three types</ins> of short-lived branches.

### Long-lived (Permanent) Branches

There will never be any commits made directly to the permanent branches

#### main

- Stores the currently released version of code and should match the code that is in production.  
- Each commit into this branch should be *tagged* with a version number in the format [Major].[Minor].[Patch]

#### develop

- Serves as the integration branch for new features. It holds the latest delivered development changes intended for the next release.

### Short-lived Branches

#### story/bug fix branches

- These are created to develop new stories or bug fixes for upcoming releases. Each new story should reside in its own branch.  
- These should be created from the *develop* branch and once complete, merged back into the *develop* branch
- These branches should follow the convention "[{ALM_Project_Code}-{ALM_Task_Number}]-{Short title}". Ex. "[ALM-223]-Add work item". Story branches not following this naming convention will fail the checks at the time of merging the pull request
- As a story branch is merged into the develop branch, the commit will be referenced in the ALM work item.

#### release branches

- Release branches support preparation of a new production release. They allow for minor bug fixes and preparation tasks before the release.
- These branches should follow the convention "release-{major}.{minor}.0" e.g. "release-1.3.0"

#### hotfix branch

- Hotfix branches are used to quickly patch production releases. They are used to addressing critical issues discovered in the main branch.
- These branches should follow the convention "hotfix-{major}.{minor}.{patch}" e.g. "hotfix-1.3.2"

## 4. Workflow

- Repository Initialization
- Story Development
- QA Story Testing
- QA Regression Testing
- UAT Release
- UAT Bug fix
- Prod Release
- Prod Hotfix

### Repository Initialization

```pikchr
scale = 0.8
fill = white
linewid *= 0.5
circle "O" fit
circlerad = previous.radius
arrow
circle "C1"
C1P: circle "C1'" at dist(O,C1) heading 30 from C1
arrow from C1 to C1P chop
Main: box height C1P.y-C1.y \
    width (C1.e.x-O.w.x)+(linewid*2) \
    with .w at 0.5*linewid west of O.w \
    behind O \
    fill 0xc6e2ff thin color gray
Develop: box same width Main.e.x - C1.w.x \
    with .se at previous.ne \
    fill 0x9accfc
"Main " rjust at Main.w
"Develop " rjust at Develop.w
```

When creating a repository the following steps should be performed

1. The default branch should be named “main”
2. Create the repo README file with the title and a short description of the project. If applicable, a boilerplate should be used and be included in the initial commit. After this initial commit, no commit can be directly made to the main branch.
3. From the main branch, create a branch named “develop”

### Story Development

```pikchr
scale = 0.8
fill = white
linewid *= 0.5
circle "C0" fit
circlerad = previous.radius
arrow
circle "C1"
//arrow right 550%//until SC3.e
circle "C2" with .e at dist(C1,C0)*3.25 right of C1
arrow
circle "C3"
arrow
circle "C4"
arrow
circle "C5"


Develop: box height 0.5 \
    width (C5.e.x-C0.w.x)+(linewid) \
    with .w at 0.5*linewid west of C0.w \
    behind C0 \
    fill 0xc6e2ff thin color gray
"Develop " rjust at Develop.w

SC1: circle "SC1" at dist(C0,C1) heading 150 from C1
arrow
circle "SC2"
arrow
circle "SC3"
arrow from C1 to SC1 chop
arrow from SC3 to C2 chop

circle "O" at dist(C0,C1) heading 330 from C0
arrow from O to C0 chop


Story_1: box same width C2.w.x - C1.w.x \
    with .nw at (C1.s.x,-0.25) \
    fill 0x9accfc
"[ALM-223]-Add work item " rjust "(story) " rjust at Story_1.w

Main: box same width Develop.e.x+dist(C0,C1)*1.5 \
    with .sw at (Develop.w.x-dist(C0,C1),0.25) \
    fill 0x9accfc
"Main " rjust at Main.w
```

When working on a new story, developers will create a new story branch from the latest commit in the develop branch.

During development of the story, if commits are made to the develop branch and development of the story will last more than 3 days, then intermittently the developer should merge the code in from develop.

Once development of the story has been completed, the latest commit from the develop branch should be merged in to the story branch and then the code should be tested on the developer's local machine.

When the story has been tested locally, a pull request (PR) should be created to merge the story branch to the develop branch. Any conflicts should be handled promptly, else the PR should be deleted.

Based on the project a peer developer will review the PR.

> Note: A PR should only be reviewed once it has passed all the checks

Then the appointed code reviewers of the project (typically the technical lead) will review the PR. If there are no concerns, the PR should be merged (**using a squash commit**) into the develop branch and the <span style="color: red">branch should be deleted.</span>

>If there are concerns, then the developer should address them with priority over any other story development

Once the PR has been merged to the develop branch, an image with the naming convention "Develop-{YYMMDDHHmms}-{ALM_CODE}" will be created and will be automatically deployed to the dev environment, plus a reference to the commit and image name will be created in the ALM work item. (Hence it is important to follow the naming conventions)

> Note: If the naming conventions are not adhered to, then the PR checks will fail.

### QA Story Testing

When the QA team member tests a story, they will deploy the relevant image to the QA environment and perform the test suites.

The image name will include the ALM work item code to make it more recognisable. The image name can be found in the ALM comments or by clicking the tick next to the commit that needs testing.

### QA Regression Testing

```pikchr
scale = 0.8
fill = white
linewid *= 0.5
circle "C0" fit
circlerad = previous.radius
arrow
circle "C1"
//arrow right 550%//until SC3.e
circle "C2" with .e at dist(C1,C0)*3.25 right of C1
arrow
circle "C3"
arrow
circle "C4"
arrow
circle "C5"


Develop: box height 0.5 \
    width (C5.e.x-C0.w.x)+(linewid) \
    with .w at 0.5*linewid west of C0.w \
    behind C0 \
    fill 0xc6e2ff thin color gray
"Develop " rjust at Develop.w

SC1: circle "SC1" at dist(C0,C1) heading 150 from C1
arrow
circle "SC2"
arrow
circle "SC3"
arrow from C1 to SC1 chop
arrow from SC3 to C2 chop

circle "O" at dist(C0,C1)*2 heading 330 from C0
arrow from O to C0 chop


Story_1: box same width C2.w.x - C1.w.x \
    with .nw at (C1.s.x,-0.25) \
    fill 0x9accfc
"[ALM-223]-Add work item " rjust "(story) " rjust at Story_1.w

Main: box same width Develop.e.x+dist(C0,C1)*1.5 \
    with .sw at (Develop.w.x-dist(C0,C1),0.75) \
    fill 0x9accfc
"Main " rjust at Main.w

circle "R3" at dist(C0,C1) heading 30 from C3
arrow from C3 to R3 chop

Release: box same width Develop.e.x - C3.c.x \
    with .se at (Develop.e.x,0.25) \
    fill 0x9accfc
"Release 1.2.0 " rjust at Release.w

```

When a release is imminent, the developer will create a release branch, following the naming conventions as stated earlier, from the relevant commit in the develop branch.

If necessary, some commits can be cherry-picked but should be ready for deployment after the next sprint cycle.

> If there is uncertainty to whether a story should be shipped in the next release cycle, it should not be merged into the develop branch or feature flags should be used.

Once a commit has been made into the release branch, an image will be made in the following naming convention "release-{Major}.{Minor}.0_{build_number}

Once ready, this new image can be deployed to the QA environment for regression testing.

> If there are any bug fixes found during the regression testing, a bug fix branch (similar to a story branch) should be created from the release branch e.g. "[ALM-143]-work item blank name fix". Once tested and reviewed, it should be merged into the release branch as well as the develop branch.

### UAT Release

Once the QA has signed off on the regression testing, only the same image used in the QA regression testing can be deployed to the UAT environment.

> Note: All project management guidelines should be followed before deployment to UAT

### UAT Bug fixing

```pikchr
scale = 0.8
fill = white
linewid *= 0.5
circle "C0" fit
circlerad = previous.radius
arrow
circle "C1"
arrow right 550% //until SC3.e
circle "C2" with .e at dist(C1,C0)*3.25 right of C1
arrow
circle "C3"
arrow
circle "C5"
arrow
circle "C6"
arrow
circle "C7"


Develop: box height 0.5 \
    width (C7.e.x-C0.w.x)+(linewid) \
    with .w at 0.5*linewid west of C0.w \
    behind C0 \
    fill 0xc6e2ff thin color gray
"Develop " rjust at Develop.w

SC1: circle "SC1" at dist(C0,C1) heading 150 from C1
arrow
circle "SC2"
arrow
circle "SC3"
arrow from C1 to SC1 chop
arrow from SC3 to C2 chop

circle "O" at dist(C0,C1)*2.75 heading 345 from C0
arrow from O to C0 chop


Story_1: box same width C2.w.x - C1.w.x \
    with .nw at (C1.s.x,-0.25) \
    fill 0x9accfc
"[ALM-223]-Add work item " rjust "(story) " rjust at Story_1.w

Main: box same width Develop.e.x+dist(C0,C1)*1.5 \
    with .sw at (Develop.w.x-dist(C0,C1),1.25) \
    fill 0x9accfc
"Main " rjust at Main.w

circle "R3" at dist(C0,C1) heading 30 from C3
arrow right 325%
circle "R4" with .c at dist(C1,C0)*2 right of R3

arrow from C3 to R3 chop

Release: box same width Develop.e.x - C3.c.x \
    with .se at (Develop.e.x,0.25) \
    fill 0x9accfc
"Release-1.2.0 " rjust at Release.w

circle "BF3" at dist(C0,C1) heading 30 from R3
arrow right 375%
circle "BF4" with .e at dist(C1,C0)*2.5 right of BF3
arrow from R3 to BF3 chop
arrow from BF4 to R4 chop
arrow from BF4 to C7 chop

Bugfix: box same width Develop.e.x - C3.e.x \
    with .se at (Develop.e.x,0.75) \
    fill 0x9accfc
"[ALM-283]-Update constants " rjust "(bug fix) " rjust at Bugfix.w

```

If any critical bugs need fixing in UAT, then a bug fix branch (the same as a story branch) should be created from the release branch that has been deployed UAT.

Once the code bug has been addresses and reviewed, the branch can be merged into the release branch. The same bug fix branch should be merged into the develop branch too. It should then be deployed to the QA environment and tested by a QA member. If the bug has been fixed the bug fix branch can be merged to develop and can be deployed to the UAT environment.

> Deploying to the UAT environment always requires the project manager to follow the necessary guidelines within the department's processes.

### Production Release

```pikchr
scale = 0.8
fill = white
linewid *= 0.5
circle "C0" fit
circlerad = previous.radius
arrow
circle "C1"
//arrow right 550%//until SC3.e
circle "C2" with .e at dist(C1,C0)*3.25 right of C1
arrow
circle "C3"
arrow
circle "C4"
arrow
circle "C5"


Develop: box height 0.5 \
    width (C5.e.x-C0.w.x)+(linewid) \
    with .w at 0.5*linewid west of C0.w \
    behind C0 \
    fill 0xc6e2ff thin color gray
"Develop " rjust at Develop.w

SC1: circle "SC1" at dist(C0,C1) heading 150 from C1
arrow
circle "SC2"
arrow
circle "SC3"
arrow from C1 to SC1 chop
arrow from SC3 to C2 chop

circle "O" at dist(C0,C1)*2 heading 330 from C0
arrow 1500%
circle "M3"



Story_1: box same width C2.w.x - C1.w.x \
    with .nw at (C1.s.x,-0.25) \
    fill 0x9accfc
"[ALM-223]-Add work item " rjust "(story) " rjust at Story_1.w

Main: box same width Develop.e.x+dist(C0,C1)*1.5 \
    with .sw at (Develop.w.x-dist(C0,C1),0.75) \
    fill 0x9accfc
"Main " rjust at Main.w

circle "R3" at dist(C0,C1) heading 30 from C3
arrow from C3 to R3 chop

Release: box same width Develop.e.x - C3.c.x \
    with .se at (Develop.e.x,0.25) \
    fill 0x9accfc
"Release 1.2.0 " rjust at Release.w

arrow from R3 to M3 chop
```

Once the necessary sign-offs have been received to deploy to production, the release branch should be merged into the main branch with a tag of the release version.

Once merged into the main branch, the release branch build can be deployed to production. The commit in main will be tagged upon successful deployment.

### Production Hotfix

```pikchr
scale = 0.8
fill = white
linewid *= 0.5
circle "C0" fit
circlerad = previous.radius
arrow
circle "C1"
//arrow right 550%//until SC3.e
circle "C2" with .e at dist(C1,C0)*3.25 right of C1
arrow
circle "C3"
arrow
circle "C4"
arrow
circle "C5"
arrow
circle "C6"


Develop: box height 0.5 \
    width (C6.e.x-C0.w.x)+(linewid) \
    with .w at 0.5*linewid west of C0.w \
    behind C0 \
    fill 0xc6e2ff thin color gray
"Develop " rjust at Develop.w

SC1: circle "SC1" at dist(C0,C1) heading 150 from C1
arrow
circle "SC2"
arrow
circle "SC3"
arrow from C1 to SC1 chop
arrow from SC3 to C2 chop

circle "O" at dist(C0,C1)*2 heading 330 from C0
arrow 1500%
circle "M3"



Story_1: box same width C2.w.x - C1.w.x \
    with .nw at (C1.s.x,-0.25) \
    fill 0x9accfc
"[ALM-223]-Add work item " rjust "(story) " rjust at Story_1.w

Main: box same width Develop.e.x+dist(C0,C1)*1.5 \
    with .sw at (Develop.w.x-dist(C0,C1),0.75) \
    fill 0x9accfc
"Main " rjust at Main.w

circle "R3" at dist(C0,C1) heading 30 from C3
arrow 200%
circle "R4"
arrow from C3 to R3 chop

Release: box same width Develop.e.x - C3.c.x \
    with .se at (Develop.e.x,0.25) \
    fill 0x9accfc
"Release 1.2.0 " rjust at Release.w

arrow from R3 to M3 chop

circle "HF3" at 0.6 heading 30 from M3
arrow 155%
circle "HF4"

arrow from M3 to HF3 chop
arrow from HF4 to R4 chop
arrow from HF4 to C6 chop

Hotfix: box same width Develop.e.x - C4.c.x \
    with .se at (Develop.e.x,1.25) \
    fill 0x9accfc
"Hotfix 1.2.1 " rjust at Hotfix.w
```

If there is a critical issue in production, then a hotfix branch should be created with the naming convention "hotfix-{major}.{minor}.{patch}".

Each commit to the hotfix branch will result in an image being built with the following syntax "hotfix-{major}.{minor}.{patch}_{build_number}"

Once the hotfix has been developed, reviewed and tested, the respective hotfix image can be deployed to the QA environment for the final testing from the development team's side. Then it should be tested in UAT.

Once the required sign-offs have been received, it can be merged into the main branch and deployed to production.

The hotfix branch should then we merged into the develop branch.
