# Total findings: 10 - 7 SAST, 3 SCA

## Real findings you would prioritize, with severity and rationale.

> The following findings were found using Semgrep Community Edition, and then the second iteration of improving the CI pipeline used Semgrep’s Pro Engine and their AI Powered Detection tooling. The findings were then validated by both ChatGPT and Cursor / Gemini. 

### P0 - fix as soon as possible (critical)

#### IDOR within the search.py file

- **Explanation**: This finding was validated by my own testing and then found by Semgrep’s AI detection tool. Not only is this finding easy to exploit, but it also is highly impactful because it gives unauthorized access to medical records not owned by that specific user. That could lead to a data breach, which could cost the company massive financial losses and create distrust for users. With all of this in mind, this would be critical to fix. 
- **Context**: The search endpoint does at least authenticate users, but it doesn’t do a check to validate if the records returned are owned by that specific user. During my testing, as Alice’s user, I was able to search Bob’s lab results with the keyword “LDL”, which is a very common blood test that anyone could possibly guess with a quick Google search.

#### IDOR within the records.py file

- **Explanation**: Similar to the other IDOR finding, I validated this and Semgrep’s AI detection tool also found this as well. The ease of exploitability and the high impact are also the same as above, so this is also critical to fix. 
- **Context**: The read_record endpoint also authenticates users, but again doesn’t validate if the requested record is owned by that user. During my testing, I confirmed that the Alice user could easily view Bob’s records simply by guessing the name and record number that was similar to Alice’s record.

#### SQL Injection within db.py

- **Explanation**: This finding was validated by my own testing and then found by Semgrep during scanning. It is an easy to exploit finding and can lead to unintentional data exposure, resulting in another possible data breach. This would be critical to fix. 
- **Context** : When the user sends over their search term, it’s concatenated into the SQL query directly with no parameterization, so the user can send over any SQL injection statements they want to pull all of the rows of the database for example.

### P1 - should be fixed during the next sprint (high)

#### SSRF vulnerability in webhooks.py

- **Explanation**: This finding was found originally by Semgrep scanning. This is a little harder to exploit because only staff users can use this endpoint, which would make it not critical, but there’s always the chance a staff user’s account  can be compromised and then it could be used to exploit internal services. Given this, this finding should be considered high.
- **Context**: When the URL from the request is passed into the response, it isn’t validated with an allowlist or strictly validated.

#### Hardcoded JWT secret in auth.py

- **Explanation**: This finding was found by both my manual review and Semgrep scanning. Considering this is easily exploitable especially if the repo is accidentally changed to public visibility vs private on GitHub, it would be easy for an attacker to forge authentication tokens to impersonate any user. Based on this, this should be considered a high vulnerability. 
- **Context**: The JWT being hardcoded into the codebase would make it easy for an attacker to forge tokens and possibly impersonate legit users.

#### JWT expiration disabled

- **Explanation**: This finding was found by my manual review and ChatGPT and Cursor, but yet wasn’t found by Semgrep. This is very easy to exploit since once you have a valid token, you have it forever. Based on this, this should be considered a high vulnerability. 
- **Context**: The JWT expiration is purposely disabled, so this could lead to token compromise and persistent access to valid tokens.

### P2 - should be fixed over the next month (medium)

#### Plaintext password storage in db.py

- **Explanation**: This finding was missed by Semgrep, but found by ChatGPT and Cursor. This is also an easy to exploit vulnerability because if the database is compromised, attackers have access to all of the user passwords and they can be used for credential reuse attacks. It is worth mentioning that since this is a sample application, and there’s only 3 users, I doubt this would be the implementation the engineering team will use in production. Given this context, this should be a medium vulnerability.  
- **Context** : The passwords for all users are stored in plaintext to be uploaded to the database, and they’re also not hashed, leading to attackers easily getting all of the user passwords to use in other applications outside of the company’s.

#### Dependency vulnerabilities - 3 total

- There were 3 dependency vulnerabilities for the requests library. Given all of them were ranked as medium in Semgrep and also require further review to determine their reachability, they could be prioritized as mediums to ensure teams focus on fixing the above critical findings. Semgrep also mentioned that the fix versions seemed not to introduce breaking changes, so this could be an easy to resolve vulnerability after the other critical fixes are in place.

## Findings you believe are false positives or acceptable in context, with rationale.

> I would rank the following findings as P3 (low) either because they’re not important to the production service at this time or they’re just being used for this challenge.

### Findings

#### USER isn’t specified in the Dockerfile

- **Explanation**: This finding was found by Semgrep scanning. This vulnerability could be acceptable given that this is a sample application and ChatGPT actually didn’t mention this as a vulnerability as well when asked to review the vulnerabilities found. Given the other critical findings that could cause more damage to the company and customers, this should be a low severity finding for now. 
- **Context**: The last command in the file runs as root because a USER isn’t specified, so an attacker could have root privileges in the container if exploited.

#### Rate limiting isn’t implemented

- **Explanation**: This finding was only found by ChatGPT, both Semgrep and Cursor missed it. The true concern here is that this could be used in combination with the plaintext user passwords and attackers could do a credential stuffing attack and it wouldn’t be stopped by rate limiting. But since the larger issue is the plaintext passwords and once that is resolved, this finding will really impact the availability of the service if a brute force attack is done. This is definitely still an issue, but based on the more critical issues and the plaintext user passwords being addressed, this could be an acceptable risk. 
- **Context**: There’s no rate limiting implemented for the service, so brute force attacks are possible.

## What your pipeline catches.

The pipeline generally does a Semgrep scan and should fail on PRs that are critical / high (ERROR severity in Semgrep) along with the custom BAC/IDOR rule. Other findings will be reported in the CI output, but in production, findings would be surfaced as PR comments and won't block PRs. The custom IDOR/BAC Semgrep rule is set to ERROR, so matching findings fail CI and block PR merge. This broken access control issue was not originally detected by both Semgrep out of the box and the quick vulnerability scan from Cursor. Semgrep’s Pro Engine did catch the IDOR, but it took additional time to get to that point by upgrading to the Pro Engine - it was quicker to create a custom Semgrep rule to detect this.

## What your pipeline does not catch.

The custom Semgrep IDOR rule was written (with AI) to attempt to generalize possible broken access control issues, specifically when endpoints fetch records with db.get_record(...) and return data without an inline ownership check. It may miss cases where authorization is enforced through helper functions. Because of these limits, this rule should be treated as a strong CI guardrail and paired with targeted testing and code review for full broken access control coverage.
Initially, I ran the API locally and did a couple tests to see if I was able to view Bob’s records as the Alice user (which was allowed), proving there’s an access control issue. I also checked to see if I could continue using the token from Alice the following day, which was also allowed. Given this, I asked Cursor to validate that these vulnerabilities may exist and to list out any additional vulnerabilities it could find. It found 4 additional findings, 2 of which weren’t critical (overly descriptive error handling and a plaintext secret that is purposely in plaintext for this challenge). The final 2 additional findings could be critical, one being possible SQL injection risk for the search query and a potential SSRF risk related to the webhook. 
These 2 possible findings aren’t necessarily fully caught (or exploited by me during initial testing) in the pipeline to truly understand if they would exist in production, so I would say they could be a possible miss. 

## Where you would invest next if this was a production service.

I would definitely plan to work with engineering teams to better understand if teams are both building services with frameworks like FastAPI without additional security considerations, or in other words, using it right out of the box and pushing it to production. It could be a great opportunity to show that building services can be easy and secure with the right guardrails in place at the beginning of the build process. In this example, the broken access control issue came from a lack of authentication and authorization checks and logic, and both can be easily mandated internally (with both secure sample code and general education sessions or guides) within the engineering org and could potentially save the company from an incredibly expensive breach if the wrong people got access to someone’s PHI. 