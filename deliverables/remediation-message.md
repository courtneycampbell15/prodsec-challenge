> I would also link the Semgrep findings from the platform to the critical vulnerabilities in case the team wants to learn more. The platform also offers auto fixes from their AI assistant for findings that will auto-create fix PRs as well if engineers want to see other alternative fix paths and quickly fix these vulnerabilities. 

Hi Engineering Lead! The Product Security team has found 3 critical vulnerabilities in the Records service that need to be prioritized ASAP, along with a few other vulnerabilities that can be addressed later on. We would love to chat in the future about improving consistency across this service via secure guardrails to avoid having engineers recreate the wheel for sensitive code changes. If engineers would rather use Claude/Cursor/preferred AI tool to find the best fix paths, feel free to use those with secure coding prompts (for example, here are some from SheHacksPurple to help). Below are the details and let us know if we can help you all get this resolved. Thank you in advance for prioritizing this to keep our customer data safe!

## P0 vulnerabilities - prioritize fixes ASAP
### Broken Access Control / IDOR within the search.py, records.py files 
- **Impact**: Authenticated users can access other users’ medical records by changing record_id for the read_record endpoint. Authenticated users can also search other users’ records by searching for common terms like “LDL” on the search endpoint. 
- **Recommended actions**: Implement ownership checks before returning requested records to the user via search or reading a singular record. The team could also centralize authorization logic. 
- **Secure code suggestion**:
For search, this line in search.py could remediate this risk:
`return {"results": db.search_records(q, owner_id=current_user.id)}`
For reading records, this line in records.py could remediate this risk:
`if current_user.role != "staff" and record["owner_user_id"] != current_user.id:`
`raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Record not found")`

### SQL injection in search.py 
- **Impact**: User input is directly put into SQL queries, which can cause unintended database exposure.  
- **Recommended actions**: Use parameterized queries. 
- **Secure code suggestion**: 
Update the query and rows variables with the following: 
`query = "SELECT * FROM records WHERE status = 'released' AND summary LIKE ?"`
`rows = connection.execute(query, ('%' + term + '%',)).fetchall()`

## Vulnerabilities to address during the next sprint
SSRF vulnerability in webhooks.py
- Impact: The service could allow attackers to fetch their controlled URLs, which could expose internal services.  
- Recommended actions: Implement allowlisting and block private or internal IP ranges.
- Secure code suggestion: Please see Semgrep for further details. 

Hardcoded JWT secret in auth.py
- Impact: The hardcoded secret can be used by attackers to impersonate any user and bypass authentication. 
- Recommended actions: Rotate tokens and move secrets to environment variables / secrets manager. 

JWT expiration disabled 
- Impact: JWT tokens are persistent, so they can be used forever and could lead to token compromise. 
- Recommended actions: Ensure expiration validation is enabled and rotate the token / remove it from the codebase. 

## Vulnerabilities to address over the next month
- Plaintext password storage in db.py 
- Dependency vulnerabilities for the requests package
