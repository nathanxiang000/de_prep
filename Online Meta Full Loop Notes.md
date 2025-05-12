## From Blind

Product Sense:
Know big tech companies and their products: google, amazon, spotify, uber, dropbox, facebook, instagram, netflix, snapchat
Think about key metrics these companies care about. Be able to give the numerator and denominator and explain your reasoning for the metrics.
Dashboard: line vs bar chart, when to use each

Prep: ChatGPT
https://www.youtube.com/watch?v=nPJKFWMiIC8&
https://www.youtube.com/watch?v=2A2UwaPaNgQ
https://www.youtube.com/watch?v=dt7OrazE6SQ

Data Modeling:
facts:
* different types of facts and their grain
* pro/con of your fact design
* don't be afraid to give more than one fact table
* identify pk, fk and relationship (1 to 1, 1 to M)
* pay attention to the grain you defined
dimensions:
* give attributes
* bridge table,  scd 2, role play, pro/con of each type

Prep: ChatGPT, Kimball 3rd edition chapter 3 and chapter 2 as reference

SQL:
Similar to screening type questions.
group by, sum(case), having, cte, where condition, left join, full outer join, coalesce, limit
Don't be afraid to use CTEs.

Prep: stratascratch SQL medium

Python:
Also similar to screening questions. Focus more on calculating metrics using python. You're allowed to use built in sort. Think about how you would store the data and what kind of data structure you would use to derive to the solution. Leetcode doesn't have these type of questions.
dict, list, tuple, sort, for loops, set

Prep: stratascratch aglorithm easy-medium with meta filter, screening questions

Ownership
5 signals they're looking, expect one to two question for each. Prepare stories for each scenario with impact and quantifiable metrics. Follow up questions will be asked if your answer doesn't give the signal they're looking for. Be ready to adapt your story to the question. Sometimes the question might not apply to you but you have to shape your existing story to match and give meaningful result. Very important interview!
https://interviewing.io/blog/how-software-engineering-behavioral-interviews-are-evaluated-meta

Final word: The questions aren't difficult but you're fighting against the clock. Getting hint is not a bad thing, using them well is a sign of learning abilities, a skill they're looking for. Practice speaking out loud. Recommend mock interviews. My interviewers were very nice and everything felt pretty smooth.

## From the Recruiter

4 interviews 
3 1 hour technical
1 30 mins interview

Behavior:
Prioritization
Autonomy 
Influence: how use data to influence decisions 
Conflict resolution 
think of  a project the roadmap the goal these are the cross functional partners and the pushback and this is the impact of the greater org
Understanding what is data engineering and why i choose it for me

3 technical interviews
they are identical 
First 10 min product sense and engagement and growth metrics 
Messager number of messages sent per hour response time active users daily weekly monthly retention and churn 
15 min data modeling
Fact dimensions tables and implement star schema data model
Attributes and relationships between tables represented
Then go into sql ETL

2 sql and 1 python question
Manipulation of data structures

SQL FOR BATCH PROCESSING
AND TRANSFORM DATA IN SNAPSHOT LINE BY LINE
NO execution line for this just pseudo code 
Explain biz logic 
Sql: subquery grouping and having
Python list strings dictionary 
1 outside product 2 meta functionalities

Timeline: Debrief on 4 interviews and result in 2-3 weeks
No need for external tools or packages just native pythonpackages

## From Red Note

### 1
第一轮BQ，说实话这强度还是挺大的，感觉没太答到面试官想要的点上，挺多次被challenge的，感觉发挥不是很好，他们非常看重他们想要collect的signal，所以尽量多准备story，每个不用deep dive特别深而是需要cover的点比较多，比如我被问到的问题
1. Most challenging work
2. Conflicting priorities and how you deal with it
3. Most ambiguous work and how you deal with it
4. Most proud work you've done.
5. Areas of yourself to grow, how you manage through it, what are the results
基本上大方向就是这5块

### 2
电面是半个小时SQL半个小时Python，一般每个部分要解3-4道题，Python都是比较基本的dictionary, data manipulation (求average）。需要跑test case， 考虑到edge cases。
	
Onsite分为四轮：Product Sense, Data Modeling, SQL/Python ETL和Behavior。
	
Product Sense会掺杂着metric design和SQL问，比Data Scientist Analytics的product问题简单，会问比较经典的product题目，比如：如果一个metric下降上升，如何diagnose。
	
Data Modeling轮会问围绕一个产品如何设计table，主要是要考虑到这个产品的不同entity，比如说uber的话，打车的人，司机，车辆都是entity。
	
SQL/Python ETL是和亚麻的BIE最不一样的一轮，会考设计metric判断一个产品成功与否，如何用dashboard来展现这些metrics （口头说，示意画个图），最后会问你怎么用Python实现data streaming。
	
Behavioral 会和DE manager聊天，一些基本的ownership问题。

