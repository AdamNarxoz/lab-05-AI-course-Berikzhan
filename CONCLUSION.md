# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own word, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say whether it changed what the agent *can do* or only what it *admits*. If no layer changed the number of correct answers, say so and explain what the layers did change instead.

Layer 1 (tool errors goes back to model). Out of 24 questions bare loop had 16 correct, 4 wrong and 4 no answer. With layer 1 it become 18 correct, 6 wrong and 0 no answer. (Template say 20 questions but in this version of lab there is 24, so I count from 24.)

This layer changed what agent can do. Without it, when region name is unknown the program just crash. With layer 1 model read the error, call list_regions and find correct name for q23 (Karagandy) and q24 (the city of Almaty). But also it make agent answer when it should stop: q21 and q22 was no answer before and now they are wrong, so wrong answers grow from 4 to 6.

Layer 2 changed nothing, calls, tokens and outcomes was same like layer 1.                                       
Layer 3 only changed what agent admit. Result was 18 / 4 / 2, so two wrong answers become "no answer", and no new correct answer appeared.

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

q18 ask by how many percentage points CPI in Jambyl fell between 2022 and 2025.
In layer 3 it finish as UNVERIFIED. Both CPI values come correctly from get_series, so lookup was fine. Mistake was in arithmetic: model calculated (16.1 - 10.6) / 100 and answered 0.55 pp. Percentage point change is just difference, 5.5, so model confused percentage points with percent. Grounding see that 0.55 is not in any tool output and send model back two times, but model never fix the expression and harness give up.

I would add check on calculate call for pp questions. If expression divide or multiply by 100, harness reject it and tell model that percentage point change is difference of two rates. It cost one or two more model calls on pp questions, plus tokens and time.
Also it add new way to fail: harness need to know question type, and this rule can block calculation which really need the 100.

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

I would choose fixed workflow. On 15 fixed-shape questions it got 13 correct and 0 wrong, when bare agent got 9 correct and 2 wrong. Workflow used 15 model calls instead of 26 and 3,852 tokens instead of 19,835 (around 5 times less). Time was 16 s instead of 47 s. When workflow fail, it give no answer, not wrong answer. Ranking is done by normal code (compare_regions and then first or last item), so model can not choose wrong end of list. For report which is published every month in same form, most important for me is cost and to never print wrong region. Only weakness I saw is old region names which model copy into JSON (q21, q22), and small alias table in code can fix it.

I would choose agent for questions without fixed shape, for example how much indicator changed between two years, because it need several lookups and calculation.

---

I wrote this file myself, without any AI tool.
