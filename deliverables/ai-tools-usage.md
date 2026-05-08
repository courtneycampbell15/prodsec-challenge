## If you used AI, where did you use it and why?

I used AI to write the custom Semgrep detection rule and complete a full check on my solution to ensure all of the deliverables were covered. Typically, when I am faced with an engineering challenge whether at work or in personal projects, I enjoy tackling them just based on past experience and research at the start. After I believe I’ve adequately answered the questions or solved the challenge I’m facing, I’ll run my solution with the resources I used through Cursor (if it’s code related), and/or with ChatGPT and Claude to see if the models differ to get a fully complete solution. In this case, I used ChatGPT and Cursor (Gemini) to look over my solutions and added in resources related to common FastAPI vulnerabilities and secure guardrails to ensure the models can reason with the correct scope to give the best solution. ChatGPT + Cursor / Gemini were extremely useful for the Semgrep detection rule since I don’t have a lot of experience with writing custom Semgrep rules. 

## What did it get right?

AI got the total vulnerabilities correct and the models were able to explain the potential impacts as well, and most of the severities were accurate especially for the critical findings. The custom Semgrep rule for IDOR detection I had both ChatGPT and Cursor (Gemini) write and both were correct, but I did have to do some tailoring with both LLMs since they did differ in their opinions on the syntax. Mainly, ChatGPT was very verbose with the syntax, meanwhile Cursor made it as simple as possible because it was able to notice that this was a take home assignment and not a production level service. 

## What did it get wrong or miss?

ChatGPT missed one of the IDOR findings within the search.py file, which further shows the importance of diversifying models, doing manual (or AI assisted) application testing, and implementing vulnerability scanners to get the full picture of vulnerabilities in your applications.  
While I was Googling to find the correct commit SHAs for the GitHub actions I used in the CI, Gemini gave incorrect SHAs, so I had to spend extra time trying to find every action’s repo to find the correct SHA.

## How did you verify or challenge the output?

I utilized a couple different models, ChatGPT and Cursor / Gemini, on top of Semgrep’s scanning and AI detection tool, to make sure my solution was as well rounded as possible. Additionally, I made sure to give context from my own research and prior knowledge to ensure the models didn’t deviate or hallucinate any vulnerabilities or possible fixes.  

## Did it change your CI design, custom rule, or triage decision?

It created the Semgrep custom rule given my fairly specific prompt, but it didn’t necessarily change my triage decision given the above IDOR finding miss. I also updated the custom IDOR Semgrep rule to ERROR vs WARNING to test to see how broken access control blocking would work. It also helped bring more context around the more critical findings that I did use to improve the remediation message for the engineering lead. For example, I further added additional fix paths to consider to make the remediation actually useful vs “here’s a problem, fix it somehow”. 