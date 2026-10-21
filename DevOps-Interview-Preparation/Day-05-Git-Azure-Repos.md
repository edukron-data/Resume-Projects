# Day 5: Git and Azure Repos — Interview Questions and Answers

## Interview questions

### Q1. Explain Git's working tree, staging area, and repository.
**Answer:**

The working tree contains checked-out files. The index, or staging  
area, defines the next commit. The repository stores commits and other  
objects. `git status` shows differences between these states, `git  
diff` shows unstaged edits, and `git diff --cached` shows staged edits.

Before committing, I inspect both staged and unstaged differences so unrelated  
edits do not enter the change accidentally. A commit records the staged  
snapshot, not every working file. Understanding these states also helps explain  
why a file can appear as both staged and modified in the same status output.

### Q2. How do fetch, pull, and clone differ?
**Answer:**

`clone` creates a local repository from an existing one. `fetch` downloads  
remote objects and updates remote-tracking references without integrating them  
into your current branch. `pull` fetches and then integrates according to merge  
or rebase settings. Inspect incoming changes before integrating shared work.

For example, I fetch before reviewing how my branch differs from the remote branch,  
then choose an intentional merge or rebase. Pulling without understanding the  
configured integration behavior can create unexpected history. I also check that  
uncommitted changes are safely preserved before switching branches or integrating work.

### Q3. How does merging differ from rebasing?
**Answer:**

Merge combines histories, optionally creating a merge commit, and preserves  
existing commit identities. Rebase replays commits onto a new base and changes their  
identities. Rebase is useful for local branch cleanup; rebasing published  
shared history requires coordination because others may depend on the old commits.

For a feature branch, rebase can make the development sequence easier to review before  
merge. For a shared integration branch, preserving history is usually more important  
than cosmetic cleanup. I explain that conflict resolution and testing are still  
necessary with either approach because history structure does not prove correctness.

### Q4. Compare GitFlow and trunk-based development.
**Answer:**

GitFlow uses long-lived branches such as main and develop with release and  
hotfix branches. Trunk-based development uses a shared mainline and short-lived  
changes supported by fast tests and feature flags. Choose according to  
release cadence, integration cost, support requirements, and team practices.

A team releasing frequently may benefit from short branches and small changes, while supporting  
several released product versions may require maintained release branches. I consider  
integration frequency and automated-test confidence. Whatever strategy is chosen, clear  
branch ownership and a defined production hotfix path prevent inconsistent release practices.

### Q5. What does cherry-pick do?
**Answer:**

`git cherry-pick COMMIT` applies a commit's change to the current branch  
as a new commit. It is useful for backporting a targeted fix without  
merging an entire feature branch. Dependencies may still be missing,  
so test the resulting behavior and record where the fix was applied.

For example, a security fix may need to reach a supported release without bringing  
in unfinished features from main. I inspect the fix's prerequisites before  
applying it. The new commit has a different identity, so I document the relationship  
and ensure later merges do not accidentally reintroduce conflicting behavior.

### Q6. What is git stash used for?
**Answer:**

Stash temporarily stores tracked working-tree and index changes; `git stash  
push -u` also includes untracked files. `apply` restores a stash while  
retaining it, whereas `pop` removes it after successful application. A stash is  
local convenience, not a durable substitute for committed, backed-up work.

I inspect `git stash list` and the selected stash's diff before restoring it,  
especially when several tasks are in progress. Restoring onto changed code can  
produce conflicts. Important work should eventually become a clear commit on  
an appropriate branch rather than remain indefinitely in a locally named stash.

### Q7. How do you resolve merge conflicts?
**Answer:**

Inspect `git status`, understand both changes, and edit the final  
intended content. Stage resolved files and continue the merge or rebase.  
Run relevant tests and review the final diff. Do not blindly choose  
ours/theirs; their interpretation also differs during rebase operations.

A conflict is a request for a human decision about the intended combined behavior.  
I review surrounding logic and related tests, not only conflict markers. After  
resolution, I inspect the final staged diff to catch duplicated code, missing  
imports, or a syntactically valid result that changes business behavior incorrectly.

### Q8. Compare reset, revert, and restore.
**Answer:**

`revert` creates a new commit that undoes an earlier change and suits  
shared history. `reset` moves a branch reference and may change the index  
or working tree depending on mode. `restore` restores file content  
from a chosen source. Destructive forms can discard uncommitted changes.

For example, reverting a published defect allows teammates to continue from shared  
history without force-pushing a rewritten branch. Restoring a file can discard  
local edits if used incorrectly. I first inspect status and preserve any unrelated  
work before selecting the command that matches the intended scope of recovery.

### Q9. What makes a good pull request?
**Answer:**

A good PR is focused, explains the problem and resulting behavior, includes  
relevant validation, and identifies material deployment or compatibility risks.  
Reviewers should be able to understand why each change exists. Link relevant  
work items and resolve feedback before merging under the repository's policy.

For a deployment change, I include what triggers it, which environment it affects,  
and how success was verified. Small focused changes help reviewers find real defects.  
PR approval should represent actual review of the current revision, so significant  
new commits may require renewed validation and review under the branch policy.

### Q10. Which Azure Repos branch policies would you configure?
**Answer:**

For protected branches, require appropriate reviewers, build validation, resolved  
review comments, and linked work items where useful. Control direct pushes, force  
pushes, and bypass permissions. Policies should enforce meaningful checks without  
making emergency procedures unworkable; document and audit exceptional bypasses.

I choose policies based on risk and team ownership, such as additional reviewers for  
infrastructure or security-sensitive paths. Required builds should validate the intended  
target integration. I also review bypass privileges because a strong policy provides  
little protection if routine contributor accounts can ignore it without accountability.

### Q11. How do squash merge and merge commit differ?
**Answer:**

Squash merge combines the PR's changes into one new target-branch commit,  
giving a compact history. A merge commit preserves the branch commits  
and records the branch relationship. Choose according to audit and  
history needs; both still require testing the final integrated result.

A squash commit often makes a feature easy to revert as one unit, while preserved  
commits can help explain intermediate design decisions. Neither approach  
guarantees a clean release. I ensure the team agrees on commit-message expectations  
and how changes are traced from work items through PRs into production.

### Q12. What should be excluded from a repository?
**Answer:**

Exclude credentials, private keys, generated binaries when published  
elsewhere, dependency caches, and machine-specific files. `.gitignore` affects  
untracked files; it does not remove already-tracked content or erase  
history. Use secret scanning and access controls in addition to ignore rules.

If generated outputs are needed for deployment, I publish them through an artifact  
store with controlled retention rather than casually adding them to source  
history. Ignore rules should support the toolchain and team workflow. Repository  
scanning catches mistakes that naming conventions and developer memory may miss.

## Scenario-based questions

### S1. A secret was committed and pushed. What do you do?
**Answer:**

Revoke or rotate the credential immediately, identify its access scope, and inspect usage logs.  
Remove it from current files and move access to an approved secret store or federated identity. Coordinate  
any history cleanup because forks and clones may retain copies. Check CI logs and artifacts for  
exposure, then add preventive scanning. History rewriting alone does not make the credential safe.

I treat the credential as compromised even when there is no immediate evidence of  
misuse. Exposure can extend to developer clones, build caches, and notification content.  
The final verification includes successful use of the replacement credential,  
rejection of the old one, and a check that normal automation no longer embeds secrets.

### S2. An incorrect commit is already on main. How would you undo it?
**Answer:**

Identify the offending commit and any dependent changes. Create a reviewed revert commit on a branch  
and validate the resulting application. For a merge commit, understand the parent relationship  
before selecting a mainline parent. Merge through the normal policy or documented incident process,  
deploy the corrected artifact, and confirm recovery without rewriting colleagues' shared history.

Before reverting, I check whether later commits depend on the change or whether a  
deployment already introduced an incompatible schema update. A source revert and  
a production rollback are related but separate operations. I explain both the code  
correction and the operational steps required to restore user-visible service.

### S3. You need one production fix from a branch containing unfinished features. What is your approach?
**Answer:**

Start from the intended production baseline and cherry-pick the smallest self-contained fix.  
Inspect dependencies and conflicts, run the affected tests, and open a targeted PR. Record the  
commit's origin and ensure the fix remains in the normal development line to prevent regression  
during a later release. Do not assume a clean cherry-pick proves functional independence.

I validate the fix against the production branch's dependency versions and configuration  
because behavior may differ from the original development branch. A targeted backport  
should have a focused description and release record. After deployment, I confirm the  
original failure is resolved and the normal development path still contains the fix.

### S4. A developer accidentally deletes a local branch. Can it be recovered?
**Answer:**

Check whether a remote branch still references the commits. Otherwise inspect `git  
reflog` to find the former branch tip and create a recovery branch at that commit. Verify  
its contents before further edits. Recovery depends on reachable objects and reflog  
retention; commits already garbage-collected may not be recoverable from that clone.

I avoid running garbage collection or cleanup while recovering because unreachable  
objects may still be useful. Once the correct commit is found, creating a  
branch makes it reachable again. If the local clone cannot recover it, I check  
another clone or remote reference before declaring the work permanently lost.
