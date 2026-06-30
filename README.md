# Links

## June 30, 2026

1. [Emerging Security Practices for AI Agents](https://www.frontiermodelforum.org/issue-briefs/emerging-security-practices-for-ai-agents/)

![](https://raw.githubusercontent.com/vidyabhandary/Links/refs/heads/main/imgs/SecurityPractices%20for%20AI%20Agents.png)

## May 26, 2026

1. [How Netflix is Using Multimodal AI to Power Video Search](https://blog.bytebytego.com/p/how-netflix-is-using-multimodal-ai)

- Specialized models consistently outperform generalists at their particular task. Therefore, Netflix runs an ensemble of specialists

- The engineering team had to solve one core challenge.

How do you take all these different outputs, produced at different time resolutions, in different formats, and merge them into one searchable index?

-  Each stage handles one concern and one concern only

-  Offline Data Fusion and Temporal Bucketing (1 second each)
This offline fusion layer is the architectural heart of the system. It handles the heavy computational work outside the real-time path, so complex data intersections never interfere with ongoing ingestion

-  Hybrid Search
A query like “Joey in the kitchen” requires two fundamentally different kinds of matching.

“Joey” is a proper noun that demands exact keyword matching. “Kitchen” is a semantic concept that benefits from vector similarity search, where the system compares the mathematical distance between the query embedding and scene embeddings stored in the index.

A keyword search alone would miss scenes labeled with related terms. Also, vector search alone would struggle with proper nouns and exact phrases. The combination of both is called hybrid search, and it consistently outperforms either approach in isolation.

-  Search-as-you-type functionality, powered by indexing partial word fragments at ingest time, surfaces frame-accurate results the moment an editor begins typing.

Linguistic stemming across multiple languages ensures that “running” matches scenes tagged with “run” or “ran,” collapsing grammatical variations into a single search intent. Fuzzy matching that tolerates character-level typos and misspellings accounts for transcription errors, so that high-value shots are never lost to minor data imperfections.

- The system uses custom aggregations to cluster outputs, such as isolating the top 5 most relevant clips of an actor per episode.

## May 18, 2026

1. [git fetch vs git pull vs git pull —rebase](https://blog.bytebytego.com/p/ep215-the-anatomy-of-an-ai-agent)

git fetch downloads remote changes and updates origin/main. Your local main does not move. Nothing in your working directory changes. That makes fetch the safest option when you want to inspect what changed upstream before integrating anything.

git pull goes one step further. It fetches first and then merges the upstream branch into your current branch. Your local commits stay intact, and Git adds a merge commit to connect the two histories.

git pull —rebase is the clean one. It starts with a fetch, but instead of merging, it reapplies your local commits on top of the updated upstream branch. The result is a linear history with no merge commit.

Fetch when you just want to see what's on the remote before deciding anything. Pull when you're on your own branch and don't mind merge commits showing up in the log. Rebase when you're cleaning up a feature branch before opening a PR and want the history to read cleanly.

## May 17, 2026

1. [TED Talsk - Secrets of elite athletes](https://www.youtube.com/watch?v=KI3WJXNhCJ8)

The two foundational pillars of this mindset that anyone can apply to their professional and personal life are:

### 1. Advanced Visualization

Elite athletes don't just look at a future goal; they "time travel" by using all of their senses—seeing, hearing, touching, and feeling emotions—to establish a future reality.

* **The Neuroscience:** Visualizing an action activates the exact same parts of the brain as actually physically performing it.
* **Application:** You can use this blueprint for success to mentally rehearse high-stakes situations, such as structuring and presenting a flawless professional pitch, executing a key interview, or working toward physical health transformations.

### 2. Deliberate Practice

True confidence is built by specific, structured hard work rather than innate gifts.

* **Focus on Fundamentals:** Base your training on the core rules and primary mechanics of your skill rather than just repeating the endpoint action.
* **Rigorous Repetition:** Commit to continuous, focused repetition to build standard excellence.
* **Stretch Beyond Comfort:** Force yourself out of your comfort zone by setting objectives just slightly past your current level of competence.
* **Seek Active Feedback:** You cannot grow in isolation; rely on external coaches, mentors, or advisors to highlight blind spots and guide improvement.

### The Philosophy on Failure

For elite performers, achievement is a continuous process of progression rather than a final destination. Winning is simply used as a benchmark to measure growth, meaning **failure is never fatal; it is merely an indicator to adjust your approach**.

You will never truly know your upper limits unless you actively push the boundaries of your current capabilities.

## May 11, 2026

1. [Winning a Kaggle Competition with Generative AI–Assisted Coding](https://developer.nvidia.com/blog/winning-a-kaggle-competition-with-generative-ai-assisted-coding/)

NVIDIA’s article describes how **three LLM agents** helped win a Kaggle churn-prediction competition by generating **600K+ lines of code** and supporting **850 experiments**. 
- The technical workflow used **EDA → k-fold baselines → feature engineering/model tuning → ensemble stacking**.
- Execution speed came from GPU-accelerated tools such as **NVIDIA cuDF, cuML, XGBoost, and PyTorch**. 
- Each experiment saved **OOF predictions** and **test predictions** as `.npy` files, enabling later stacking, hill climbing, and meta-modeling.
- The final winning model was a **four-level stacked ensemble of 150 models selected from 850 experiments**. 

## Key technical points

* **LLM agents used:** GPT-5.4 Pro, Gemini 3.1 Pro, and Claude Opus 4.6. 
* **Task:** Binary classification for telecom customer churn; metric was **AUC**.
* **Modeling approach:** k-fold pipelines using models like **XGBoost, GBDT, neural networks, and other ML models**.
* **Prediction artifacts:** Saved as `train_oof_[MODEL]_[VERSION].npy` and `test_preds_[MODEL]_[VERSION].npy`.
* **Advanced techniques:** **Feature engineering, model tuning, pseudo-labeling, knowledge distillation, hill climbing, and stacking**.
* **Main technical takeaway:** LLMs accelerated code generation, while GPUs accelerated experiment execution; the advantage came from shortening the full ML experimentation loop.

## May 06, 2026

1. [Awesome Codex Skills](https://github.com/ComposioHQ/awesome-codex-skills/tree/master)

## Apr 30, 2026

1. [RAG Approaches](https://pub.towardsai.net/10-practical-rag-approaches-what-is-actually-useful-and-when-to-use-each-one-fde0d313487f)


## Apr 28, 2026

1. [How Stripe Detects Fraudulent Transactions Within 100 ms](https://blog.bytebytego.com/p/how-stripe-detects-fraudulent-transactions)

- The earlier XGBoost and DNN were good in Radar
But could not use transfer learning and took too long to experiment
- The core idea, sometimes called “Network-in-Neuron,” splits computation into multiple distinct branches, where each branch functions as a small neural network on its own. 
- Usage of embeddings - Embeddings enable geographic transfer of fraud knowledge
- Precision vs Recall for different businesses
- Explanability - The explanation layer transforms Radar from a black box into something merchants can actively collaborate with
- Moving to Production - A model that performs better on aggregate metrics might still cause a spike in block rate for smaller businesses, which would be disruptive for those merchants and their customers. Before releasing any model, Stripe measures the change it would cause to the false positive rate, block rate, and authorization rate on both an aggregate and per-merchant basis. If a model would cause undesirable shifts for certain users, they adjust it for those segments before release
  
2. [How I Built ML-Powered LLM Routing with <5ms Latency](https://pub.towardsai.net/how-i-built-ml-powered-llm-routing-with-5ms-latency-e81476a47231)

Manually deciding which local LLM to use.

Coding question? Load Qwen. Math problem? Switch to a reasoning model. General question? Back to something smaller and faster. Every switch meant waiting 30–60 seconds for the model to load into VRAM. It was exhausting.

So I built a router that does it automatically — classifies every query in under 5ms and routes it to the optimal model based on benchmark scores, historical performance, and user feedback.

The git link
https://github.com/al1-nasir/LocalForge.git

3. [Architecture code review using document chunks and Github PR](https://pub.towardsai.net/how-i-automated-architecture-code-reviews-using-agentic-ai-and-github-actions-e8f349679d81)

## Apr 27, 2026

1. [LLM Parameters Cheatsheet](https://pub.towardsai.net/the-complete-llm-parameters-cheatsheet-2026-5ae47bb14ef3)


Different parameter settings for
- RAG / Factual Q&A with citations
- Classification (e.g., intent, sentiment, safety)
- JSON / structured extraction from messy text
- SQL generation
- Code generation
- Chatbot / customer support
- Creative writing (stories, poems, marketing copy)
- Brainstorming / ideation
- Summarization (factual, extractive)
- Translation
- Agent / tool-use loop
- Reasoning / math / hard problems
- Production pitfalls (and how to avoid them)

## Apr 15, 2026

1. [How Roblox Uses AI to Translate 16 Languages in 100 Milliseconds](https://blog.bytebytego.com/p/how-roblox-uses-ai-to-translate-16)
Some learnings

- Mixture of Experts
- Knowledge Distillation (sometimes described as a teacher-student approach)
- Iterative backtranslation
- Embedding cache between encoder and decoder

## Apr 2, 2026

1. [What Claude Code’s Source Leak Actually Reveals](https://medium.com/@marc.bara.iniesta/what-claude-codes-source-leak-actually-reveals-e571188ecb81)

2. [The Claude Code Source Leak: fake tools, frustration regexes, undercover mode, and more](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/)

3. [Python rewrite](https://github.com/ultraworkers/claw-code)

---

## Mar 31, 2026

1. [The 15-File Setup That Turned Claude Code Into My Development Team](https://pub.towardsai.net/the-15-file-setup-that-turned-claude-code-into-my-development-team-04bc38da49de)

---
## Mar 27, 2026

1. [Stanford CS221: Artificial Intelligence](https://www.youtube.com/playlist?list=PLoROMvodv4rMeDqwS1yFl3j3sR_-MQNEN)
---

## March 25, 2026

1. [Parquet shrinks data 100x](https://blog.dataexpert.io/p/parquet-can-shrink-your-data-100x)
2. [Context Engineering is a skill](https://pub.towardsai.net/context-engineering-is-a-skill-most-developers-are-skipping-it-9938678292b8)

Many github repos for skills

3. [Anti-gravity Skills](https://github.com/sickn33/antigravity-awesome-skills/)
4. [Architect Review Skill](https://github.com/sickn33/antigravity-awesome-skills/blob/main/skills/architect-review/SKILL.md)
5. [Avoid AI Writing](https://github.com/sickn33/antigravity-awesome-skills/blob/main/skills/avoid-ai-writing/SKILL.md)
6. [Backend Architect](https://github.com/sickn33/antigravity-awesome-skills/blob/main/skills/backend-architect/SKILL.md)
7. [Comprehensive Code Review](https://github.com/sickn33/antigravity-awesome-skills/blob/main/skills/comprehensive-review-full-review/SKILL.md)

---

## November 28, 2025

1. [Networks Explained](https://pub.towardsai.net/networking-explained-from-whats-network-to-ai-powered-networks-827a1bb83b7c)
2. [Prepping for study using AI - Part 1](https://www.aifire.co/p/1-hour-of-this-ai-prep-will-save-you-100-hours-of-study-part-1)

#### 1 - 
> "I am a complete beginner who wants to learn [Topic: e.g., Video Editing with CapCut]. My specific goal is to [Goal: e.g., create a 1-minute travel vlog for TikTok with music and transitions]. Act as an expert researcher. Please search Reddit, specialized forums, and learner communities to find the '80/20' resources. I want to find the top 3 most recommended tutorials or free courses that teach the fundamentals quickly. For each recommendation, explain why people like it and list one negative thing about it."

> Why this prompt works:

> "Reddit and forums": It looks for real human discussions, not marketing blogs.

> "80/20 resources": It asks for the 20% of materials that give 80% of the results.

> "One negative thing": This helps you trust the result. No course is perfect.

#### 2 - 
> "I am about to study this document. I need you to create a 'Mental Map' for me. Please list the 5 most critical concepts I must understand to master this topic. For each concept, provide a simple 1-sentence definition. Also, list the 10 most common jargon words I will encounter. Do not give me long explanations yet. I just want the structure."

#### 3 - 

> "Based on the text I uploaded, create a 10-question multiple-choice quiz about the most important ideas. I want to test my knowledge before I start studying. Please provide the questions, but do not show me the answers until I ask for them."


3. [Prepping for study using AI - Part 2](https://www.aifire.co/p/1-hour-of-this-ai-prep-will-save-you-100-hours-of-study-part-2)


#### 1 - 
> "I have pasted the transcript of a long video lecture below. Please rewrite this information as a structured textbook chapter. Use clear headings for each main topic. Use bullet points for key details. Bold the most important terms. Remove all the speaker's 'umms', 'ahhs', and off-topic jokes. Make it scannable so I can read it in 10 minutes."

#### 2.

> The Power Of "Interleaving"
> Most people study like this:

- Monday: 3 hours of Spanish.
- Tuesday: 3 hours of Coding.

> This is boring and inefficient. Your brain gets numb doing the same thing for too long.

> Science suggests a method called Interleaving. This means mixing subjects.

> Monday: 45 mins Spanish, then 45 mins Coding, then 30 mins Spanish.

Why does this work? When you switch from Spanish to Coding, your brain has to "reset." It has to work hard to recall the Spanish words again when you switch back. This effort strengthens the memory. It also keeps you from getting bored.

4. [Fine-Tune an Open Source LLM with Claude Code/Codex (Hugging Face Model Trainer Skill)](https://www.youtube.com/watch?v=HGPTUc7tEq4)
5. [Using Coding Assistants - Beyond Vibe Coding](https://blog.tedivm.com/guides/2026/03/beyond-the-vibes-coding-assistants-and-agents/)

---

## November 16, 2025

1. [GPT Hacks](https://www.aifire.co/p/4-secret-chatgpt-hacks-that-actually-cut-my-work-time-by-50)

> 1. The "Magic" Step. "Great. Now, look back at our whole conversation. Write one single prompt that, if I used it at the beginning, would create this final perfect response in one go."

> 2. Save and test your new prompt. 
ChatGPT will now give you a "golden" prompt. It might look like this:
Now, open a new chat, paste this "golden" prompt in, and watch what happens. It should create the perfect result immediately.

> 3. The 'Other Side' Trick: First, you ask ChatGPT to create something for you. And this is the important part, you immediately ask it to "switch sides" and act as a tough critic.

> 4. Flip the script
Now is the time for magic. Ask ChatGPT to play the role of your toughest audience and "attack" it.

> 5. The 'Blueprint' Trick: Plan Before You Build
The 'Blueprint' trick solves this by forcing ChatGPT to explain its step-by-step reasoning before it gives the final result.
It is like finalizing the blueprint (plan) of a house before actually building the house.

---

## August 30, 2025

1. [Dependency Injection](https://www.youtube.com/watch?v=yunF2PgJlHU)
2. [Good System Design](https://www.seangoedecke.com/good-system-design)
3. [Good API Design](https://www.seangoedecke.com/good-api-design)

---

## July 19, 2025

1. Pirates of the RAG 
[Pirates of the RAG: Adaptively Attacking LLMs to Leak Knowledge Bases](https://arxiv.org/abs/2412.18295)

**"Oops ! I overshared !!!"** Looks like this problem is not unique to humans ! With strategic questioning, RAG based AI spills its secrets !

2. [Refactor of a legacy service to reduce cloud spending](https://blog.duolingo.com/reducing-cloud-spending/) 

I recently completed my 1.5-year streak on Duolingo (just 5 minutes a day—let’s call it dabbling rather than mastery 😅. Yes my Duolingo app icon is on fire !!!)

It feels almost serendipitous to come across this article about how Duolingo slashed cloud costs. One jaw-dropping stat? After the refactor of a legacy service, they uncovered 2.1 billion unnecessary API calls… every single day! 😱

3. [Concurrency Bug but a calm weekend](https://pushtoprod.substack.com/p/netflix-terrifying-concurrency-bug) 

Unconventional technique for a peaceful weekend with a chaos-monkey type self-healing, pragmatic engineering on a Friday afternoon.

How?

- Resizing cluster to meet max capacity
- Randomly kill a few instances every 15 minutes 

Takeaway - Sometimes production incidents can be problem solved to align with quality of life.

4. [HBR: How Generative AI will change Sales](https://hbr.org/2023/03/how-generative-ai-will-change-sales) 

Of all the techno-functional terms I have across this one stands out as remarkably unconventional - "Boundary Spanner".

A “boundary spanner”—an individual who understands and is respected by technical experts as well as by sales team members.
Harvard Business Review

---

## June 10, 2025

1. DNS resolution 
[Let's Understand DNS For Real](https://www.danielfullstack.com/article/dns-does-not-have-to-be-hard)

---

## March 09, 2025

1. **NTP and PTP Explained** 
    [How Computers Synchronize Their Clocks - NTP and PTP Explained](https://www.youtube.com/watch?v=WX5E8x3pYqg)

2. **The Obscure System That Syncs All The World’s Clocks** (March 09, 2025)
    [NTP - The Obscure System That Syncs All The World’s Clocks](https://www.youtube.com/watch?v=CwZW0CO7F-g)

3. **NTP vs PTP** [NTP vs PTP](https://www.youtube.com/watch?v=lOUqOEkDT5I)

---

## December 15, 2024

1. **Browser Rendering Process** 
    [Insightful and detailed article on the browser rendering process](https://abhisaha.com/blog/exploring-browser-rendering-process/)

![Pictorial representation](https://raw.githubusercontent.com/vidyabhandary/Links/refs/heads/main/imgs/rendering.jpg)

    TL;DR (From the article)

1. DNS Lookup: When a URL is entered, the browser performs a DNS lookup to convert the domain name into an IP address, allowing it to locate the website’s server.

2. TCP/TLS Handshake: The browser initiates a TCP handshake to establish a connection with the server. If the site is secure (HTTPS), a TLS handshake is also conducted to encrypt data transmission.

3. HTTP Request/Response Cycle: After establishing a connection, the browser sends an HTTP request for the website’s content, and the server responds with the necessary HTML, CSS, JavaScript, and other assets.

4. Tokenization: The browser reads the HTML response as raw data, converting it into individual characters and then tokens (e.g., <html>, <body>), which help the browser understand the document’s structure.

5. DOM Tree Creation: The browser builds the DOM (Document Object Model) tree, a hierarchical representation of the HTML document’s structure, with each node representing an element or text content.

6. CSSOM Tree Creation: The browser parses the CSS to create the CSSOM (CSS Object Model) tree, which represents the styles associated with the HTML document’s elements.

7. Render Tree Creation: The DOM and CSSOM trees combine to form the Render Tree, a visual representation of the page’s layout that includes only visible elements and their computed styles.

8. Layout: The browser calculates the exact size and position of each element on the screen, based on CSS properties like margins, padding, and positioning (e.g., static, absolute).

9. Painting: Using the Render Tree, the browser paints pixels onto the screen, filling in colors, images, borders, and shadows as defined by the CSS styles.

---

## December 10, 2024

1. **Deletes are difficult** 
    [Deletes are difficult](https://notso.boringsql.com/posts/deletes-are-difficult/)

---

## December 2, 2024

1. **Binary vector embeddings and Matryoshka embeddings** (Dec 02, 2024)
    [Binary vector embeddings are so cool](https://emschwartz.me/binary-vector-embeddings-are-so-cool)

2. **Prioritizing sanity** 
    [How We Built a Self-Healing System to Survive a Terrifying Concurrency Bug At Netflix](https://pushtoprod.substack.com/p/netflix-terrifying-concurrency-bug)

---

## November 16, 2024

1. **45 ways to break an API server** 
    [45 ways to break an API server(negative tests with examples)](https://dev.to/zvone187/45-ways-to-break-an-api-server-negative-tests-with-examples-4ok3)
2. **Cuckoo hashing Visualizations** 

- [Interactive Cuckoo](https://itu.dk/people/maau/teaching/visualisation/cuckoo-hashing/interactive.html)
- [Cuckoo Hashing](https://itu.dk/people/maau/teaching/visualisation/cuckoo-hashing/index.html)
- [Another Cuckoo Hashing visualization](https://www.lkozma.net/cuckoo_hashing_visualization/)
3. **Claude Artifacts** 

- [Claude Artifacts](https://simonwillison.net/2024/Oct/21/claude-artifacts)
- [Claude transcript for Extracting links from pasted HTML](https://gist.github.com/simonw/0a7d0ddeb0fdd63a844669475778ca06)
4. **Accountability sinks** 
    [Nobody is responsible](https://aworkinglibrary.com/writing/accountability-sinks)
5. **Understanding Round Robin DNS** 
    [Round robin DNS visualization](https://blog.hyperknot.com/p/understanding-round-robin-dns)
6. **Art of attention** 
    [Concentration](https://billwear.github.io/art-of-attention.html)

---

## October 4, 2024

1. **B-trees and database indexes** 
    [B-trees and database indexes](https://planetscale.com/blog/btrees-and-database-indexes)
2. **B+ tree visualiztion** 
    [B+ tree visualiztion](https://bplustree.app/)
3. **B tree visualiztion** 
    [B tree visualiztion](https://btree.app/)
4. **Bitten by Unicode** 
    [Bitten by Unicode](https://pyatl.dev/2024/09/01/bitten-by-unicode/)
5. **Neuroscientist: How to LEARN ANYTHING Without Any Effort** 
    [Youtube - Some strategies for learning more effectively](https://www.youtube.com/watch?v=I2dm72OuK6M)

**Key takeaways**

- Pause for 10 seconds
- Focus and Rest
- Every 90 min have 10 breaks of 20-30 seconds break, randomly (this is the key)

---

## September 13, 2024

1. **SaaS Pricing Models** 
   [SaaS Pricing Models](https://www.cobloom.com/blog/saas-pricing-models#)
2. **One Minute Focus** 
   [One Minute Focus](https://oneminutefocus.com/)

---

## August 31, 2024

1. **Database Caching** 
   [Database Caching](https://www.prisma.io/dataguide/managing-databases/introduction-database-caching)

---

## August 25, 2024

1. **8 versions of UUID and when to use them** 
   [https://ntietz.com/blog/til-uses-for-the-different-uuid-versions](https://ntietz.com/blog/til-uses-for-the-different-uuid-versions)

   Key takeaway: UUIDs have 8 versions, each with specific use cases. V4 (random) and V7 (sortable) are most common.
2. **Reverse Engineering TicketMaster's Rotating Barcodes (SafeTix)** 
   [https://conduition.io/coding/ticketmaster](https://conduition.io/coding/ticketmaster)

---