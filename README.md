### **SU-CPSC 5710 25FQ Notes** [Canvas Link](https://seattleu.instructure.com/courses/1623416)
---
***Quizzes Notes***
- [QUIZ1](QUIZ1.md)
- [QUIZ2](QUIZ2.md)
- [QUIZ2_EXTRA](PREVENT_SIN.md)

***COMMON EXAM QUESTION TYPES***
**Type 1: "Exploit This Code"**
- You're given vulnerable code
- Asked how to attack it
- Must explain step-by-step exploitation
- Example: SQL injection payload with explanation

**Type 2: "Fix This Vulnerability"**
- Given vulnerable code
- Write corrected version
- Explain why your fix works

**Type 3: "Identify the Sin"**
- Given scenario or code
- Identify which vulnerability it is
- Explain why

**Type 4: "Real-World Impact"**
- Given attack scenario
- Explain consequences
- Suggest preventions
- Example: Describe DAO hack implications

**Type 5: "Practical Implementation"**
- Write code to prevent vulnerability
- Example: Prepared statement for SQL
- Input validation function
- Secure hashing implementation

***EXAM SUCCESS TIPS***
1. **Read Questions Carefully**
   - Identify exactly what's being asked
   - "How to exploit" vs. "how to prevent" are different

2. **Show Your Work**
   - Explain your reasoning
   - Partial credit for right approach, wrong execution
   - Demonstrate understanding, not just memorization

3. **Use Examples**
   - Reference real attacks (DAO, BA, etc.)
   - Concrete examples are stronger than abstract explanations

4. **Code Legibly**
   - Comment your code
   - Use clear variable names
   - Proper formatting shows competence

5. **Time Management**
   - Easy questions first
   - Don't get stuck on one problem
   - Review at end if time permits

6. **Focus on "Why"**
   - Not just "how to exploit"
   - Understanding WHY it works shows mastery
   - Prevents confusion on related topics


***Recommended Study Approach (From Guide.pdf)***

**Step 1: Do All Practice Problems**
- Go through each exercise in the lecture
- Don't just read solutions—actually code them
- Try to exploit vulnerabilities yourself
- Understand why each attack works

**Step 2: Take Practice Tests**
- Create your own practice test from lecture material
- Include questions about:
  - How each vulnerability works
  - Why it's dangerous
  - How to prevent it
  - Real-world examples
- Time yourself to match exam conditions

**Step 3: Mimic Test Environment**
- Find quiet location
- Set timer for allocated test time
- Complete practice test without references
- Review what you got wrong immediately
- Do this 1 week before actual exam

**Step 4: Focus Areas for This Exam**

**Know These Cold:**
1. Buffer Overflows - memory layout, overwriting return addresses
2. SQL Injection - how to craft payloads, prepared statements
3. XSS - stored vs. reflected, why it works
4. Format Strings - how to leak memory with %x, %s
5. Command Injection - shell metacharacters, execution with program privileges
6. Key concepts - why each vulnerability is dangerous, not just how to exploit

**Practice These Exercises:**
- Buffer overflow on onlinegdb
- Command injection (C and Python)
- SQL injection (both tasks)
- XSS (stored and reflected)
- Format string memory leak
- Integer overflow in Solidity
- All concepts from Sins 7-19

**Step 5: Create Study Materials**

**Make Flash Cards:**
- Front: "What is SQL Injection?"
- Back: "Attacker inserts SQL code into input fields, bypasses WHERE clause. Example: ' OR '1'='1"

**Create Concept Maps:**
- Center: Vulnerability name
- Branches: How it works, why it happens, how to prevent, real examples

**Write Summaries:**
- One paragraph per sin
- Include: definition, exploitation method, prevention, real-world impact
---
