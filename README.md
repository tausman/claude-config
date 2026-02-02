# Usage

This pulls from agents & skills introduced here: https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md -- specifically [this section](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md#what-works-even-better-frequent-intentional-compaction).

Also pulled from https://github.com/carterbs/agent-config

This is a WIP to accomplish larger pieces of amibiguous work with agents. There are likely improvements that can be made. The goal is to start simple, experiment, understand & improve the workflow.

Copy these agents/skills into the respective Claude agents/skills directories: `~/.claude/{agents,skills}`

## Research
Start with a clear context. Use the `/research-codebase` skill which will spawn agents to understand the problem space. You can create a `PROMPT.md` file along the lines of **"I want to understand X, can you give me a high level overview of the important components & how this particular section Y works. Write your findings to `RESEARCH.md`"**. You can continue to add to the research by asking follow up questions. If this is a multi-service or multi-repo change, it can be worth having multiple RESEARCH.md files.

For ambigious changes, this research phase is just as important for the human in the loop as it is for the agent. This part should require the most brainpower. Being lazy here will lead to not understanding the plan or implementation & will likely lead to poor results.

## Plan
Clear the context. You should have a better idea of the ambiguous parts you did not understand before. Use the `/create-plan` skill & add the `RESEARCH.md` file(s) to the context. Describe the outcome that you want to achieve. Add pictures/diagrams if you can. If you know specific areas of code or components that need to be modified, call that out. If this is a multi repo or multi service change, be sure the agent has access to all the code. Be sure to tell the agent write the plan to a `PLAN.md` file.

Additionally, prompt the agent with something like **"Break the task down into phases. Each phase should be a commit. Before a commit is made, new tets should be added for the new code paths. Before a commit is made, run all tests to make sure they are passing. If tests are failing, keep iterating until they succeed. If you cannot run tests or they are failing, stop & ask for guidance."** This makes it so that the agent has small pieces of work & a feedback loop.

Review the plan. You can either iterate on the plan or start over if things do not look right. It is challenging to modify the plan once the agent starts implementation, so you want this to be as accurate as possible. I also tell the agent to do work for each repo in a separate worktree. That way I can juggle a few tasks at the same time, or pause things & work on something else without disrupting progress.

## Implement
The final step is to clear the context, "accept all edits" & `/implement-plan`. If the agent is working in a worktree, I will open the worktree & follow along to see how things are going. If the code/problem is not well understood, or there are a lot of phases in the plan, it is likely that there will be some human intervention.

You can pause the main agent, start a new session in the worktree, investigate code changes and ask the new agent for assistance. Then, inform the main agent of changes, tell it to update the plan (add/edit a phase) and have it continue where it left off.

I have found that if there are large changes, modifying the plan live is not always effective. If live modification is not working, the two approaches you can take is: take your learnings & try again from step 1 or 2, or let the agent complete its work & clean up afterwards -- with the help of additional agents.

If you don't get the desired results, there is nothing wrong with going back to the planning stage or even research stage. Part of software development is learning by doing & agentic development just lets us do this much faster. We can find pitfalls or missed edge cases within hours instead of days.

## PRs
You should have a chain of commits for each phase. The commits should be small & all tests should be passing. Each commit can be turned into an easy to review PR in a PR stack.

You can assign each commit a branch name:
```bash
git brach -f <branch_name> <commit> && git push origin <branch_name>
```

& create a pr:
```bash
gh pr create --title "$(git --no-pager log -1 --pretty=%s)" --head $(git branch --show-current) --base <base_branch> --draft
```

If this is a larger change, there will likely be some tuning necessary or PR feedback. You can use the follwing to interactively `edit` each commit.
```bash
git rebase -i --update-refs <first_commit>^
```

Then
```bash
git push origin -f <branch_name_1> <branch_name_2> ...
```

There are other tools to do this such as `jj`, `git-machete`, `graphite`, but built-in git commands also work.

## Artifacts
At the end, you should have `PROMPT.md`, `RESEARCH.md`, `PLAN.md` & a PR stack. These artifacts can be persisted with a Jira ticket -- or even better VCS -- so that things are self documented for human & agent use in the future.

## TODO
- See if `jj` makes the workflow for creating a PR stack easier
- Additional agent to make commits & PRs more ergonomic
- Better testing loops built into the planning phase
- Figure out how/why handoff


