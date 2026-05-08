# AI-Assisted PR Security Reviewer

> Summary: The reviewer would take in the findings from Semgrep and use the additional context from the application’s code to further improve the context around the findings which will in turn help determine the severity of the findings. It should not be blocking PRs based on just its own understanding - it is an assistant for a reason. 


## Inputs

Describe what context the reviewer should receive.

- **Diff**: only code changes within the PR will be considered during the review process - to keep the review fast, the entirety of the repo shouldn't be scanned 
- **Route map**: API endpoints from the main.py and routes/*.py files
- **Auth model notes**: Users should only be accessing their own records unless the user has the staff role. The assistant should also check the auth.py file and ownership checks/roles to understand the authorization/authentication structure. 
- **Scanner findings**: Semgrep output with the custom rules folder as well, along with any failing unit or integration tests that could impact security 
- **Repo / service ownership**: To make it easier on the ProdSec team and the assistant, if the engineering org uses codeowners files or has an inventory management tool, this could help assign the assistant's confirmed findings to the correct team or engineering manager.  


## Outputs

Describe the reviewer's output format.

> The assistant should output its review in the PR, but also create a Jira ticket with the relevant information formatted correctly, along with the ticket being assigned to the ProdSec team to review. As soon as the ProdSec team is good with the assessment, the Jira ticket could be reassigned to the correct engineering team to fix it.  

- **Risk label**: Critical, High, Medium, Low 
- **Finding summary**: 2 - 3 sentences that details what the PR covers and what it may introduce as a risk and what it could impact. It must be very focused on keeping this short and sweet. 
- **Confidence**: High, Medium, Low
    - Could also pull the confidence context from Semgrep API to consider during the assistant review
- **Suggested owner**: ProdSec team should review first, but there should be a mention of what team owns this possible vulnerability given the codeowners file or the inventory management tool.  
- **Block/comment/escalate decision**: 
> Blocking should be done after a period of testing, not necessarily at the start of implementation.
    - **Block**: if the PR contains any hardcoded secrets or if there’s an obvious IDOR issue as an example should be blocked.
    - **Comment**: probably the most used decision, the reviewer should always leave comments on each PR 
    - **Escalate**: make take some focused testing time, but if the the assistant can’t necessarily determine if the findings it has found are reproducible like if it doesn’t have enough context for an authorization workflow, it should escalate to ProdSec and attach its notes.   


## Guardrails

Describe how the system avoids unsafe or noisy behavior.

- **No blocking on low-confidence AI-only findings**: The assistant should not block a PR without its own deterministic evidence or at least a Semgrep finding. 
- **Require deterministic evidence for PR failure**: The assistant must have a tangible example that this vulnerability is real, such as it agrees with Semgrep, there’s security related unit tests failing, there’s a known CVE, or it can show the path of exploitation for the vulnerability. It must only check code within the PR and not imagine insecure code from its own memory to block a PR. 
- **Detect likely false positives**: The assistant should at least detect if the PR has later commits that fix the vulnerability, avoid reviewing test code and blocking the PR based off that, and adjust its confidence level if this does happen. 
- **Avoid sending secrets to third-party models**: Implement  tooling (open source possibly) to redact secrets from prompts, avoid reviewing the .env file if it's accidentally committed, avoid secrets accidentally added historically in the repo, and reviewer will be built with models approved internally (private or self hosted as an option). 
- **Prompt injection protection**: If the PR has text that attempts to override the policy we have created, it should mention that in its review and say it’s not relevant.

## Evaluation

Describe how you would measure whether the reviewer is working.

> Metrics should include context around the usefulness of the assistant’s reviews and how quickly it can do its reviews. We could also test models against each other during the early stages of testing to see what may perform better depending on if we value accuracy vs speed. 

- **Seeded test cases**: There should be multiple fake PRs that introduce vulnerabilities, like IDORs, avoiding authentication or authorization checks in code, or possible SSRF vulnerabilities. The goal here is to make sure vulnerable code is caught by the assistant and is ranked correctly severity wise. This could be specific to the repo or we could develop a guide that covers a lot of different vulnerable patterns, similar to the OWASP Juice Shop app structure.  
- **False positive rate**: The ProdSec team measures when they review the assistant’s findings and determine they’re either not actionable or inaccurate. This could also further be tested if engineers ignore the review and suggestions as well. 
- **Missed broken access control variants**: Even during this assignment, there was a worry that the broken access control I found wasn’t going to be found by ChatGPT or Cursor and luckily it somewhat was (1 out of 2 of the issues was found). This is really pushing the importance of tool diversity: AI + Semgrep scanning in CI and possible manual testing done by the ProdSec team. 
- **Engineering feedback**: Feedback should be considered across the ProdSec team and engineering teams. We should consider if the AI comments and fixes are actually helpful and succinct vs unhelpful and noisy. The other critical part is speed: engineers shouldn’t be waiting over a minute for an AI assistant to review their code - they have a lot of features to push out quickly for customers. 
