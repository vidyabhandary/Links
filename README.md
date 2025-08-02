# Links

## August 25, 2024

1. **8 versions of UUID and when to use them** 
   [https://ntietz.com/blog/til-uses-for-the-different-uuid-versions](https://ntietz.com/blog/til-uses-for-the-different-uuid-versions)

   Key takeaway: UUIDs have 8 versions, each with specific use cases. V4 (random) and V7 (sortable) are most common.

---

2. **Reverse Engineering TicketMaster's Rotating Barcodes (SafeTix)** 
   [https://conduition.io/coding/ticketmaster](https://conduition.io/coding/ticketmaster)

---

## August 31, 2024

3. **Database Caching** 
   [Database Caching](https://www.prisma.io/dataguide/managing-databases/introduction-database-caching)

---

## September 13, 2024

4. **SaaS Pricing Models** 
   [SaaS Pricing Models](https://www.cobloom.com/blog/saas-pricing-models#)

---

5. **One Minute Focus** 
   [One Minute Focus](https://oneminutefocus.com/)

---

## Oct 4, 2024

6. **B-trees and database indexes** 
   [B-trees and database indexes](https://planetscale.com/blog/btrees-and-database-indexes)

---

7. **B+ tree visualiztion** 
   [B+ tree visualiztion](https://bplustree.app/)

---

8. **B tree visualiztion** 
   [B tree visualiztion](https://btree.app/)

---

9. **Bitten by Unicode** 
   [Bitten by Unicode](https://pyatl.dev/2024/09/01/bitten-by-unicode/)

---

10. **Neuroscientist: How to LEARN ANYTHING Without Any Effort** 
    [Youtube - Some strategies for learning more effectively](https://www.youtube.com/watch?v=I2dm72OuK6M)

**Key takeaways**

- Pause for 10 seconds
- Focus and Rest
- Every 90 min have 10 breaks of 20-30 seconds break, randomly (this is the key)

---

## Nov 16, 2024

11. **45 ways to break an API server** 
    [45 ways to break an API server(negative tests with examples)](https://dev.to/zvone187/45-ways-to-break-an-api-server-negative-tests-with-examples-4ok3)

---

12. **Cuckoo hashing Visualizations** 

- [Interactive Cuckoo](https://itu.dk/people/maau/teaching/visualisation/cuckoo-hashing/interactive.html)

- [Cuckoo Hashing](https://itu.dk/people/maau/teaching/visualisation/cuckoo-hashing/index.html)

- [Another Cuckoo Hashing visualization](https://www.lkozma.net/cuckoo_hashing_visualization/)

---

13. **Claude Artifacts** 

- [Claude Artifacts](https://simonwillison.net/2024/Oct/21/claude-artifacts)

- [Claude transcript for Extracting links from pasted HTML](https://gist.github.com/simonw/0a7d0ddeb0fdd63a844669475778ca06)

---

14. **Accountability sinks** 
    [Nobody is responsible](https://aworkinglibrary.com/writing/accountability-sinks)

---

15. **Understanding Round Robin DNS** 
    [Round robin DNS visualization](https://blog.hyperknot.com/p/understanding-round-robin-dns)

---

16. **Art of attention** 
    [Concentration](https://billwear.github.io/art-of-attention.html)

---

17. **Binary vector embeddings and Matryoshka embeddings** (Dec 02, 2024)
    [Binary vector embeddings are so cool](https://emschwartz.me/binary-vector-embeddings-are-so-cool)

---

## Dec 02, 2024

18. **Prioritizing sanity** 
    [How We Built a Self-Healing System to Survive a Terrifying Concurrency Bug At Netflix](https://pushtoprod.substack.com/p/netflix-terrifying-concurrency-bug)

---

## Dec 10, 2024

19. **Deletes are difficult** 
    [Deletes are difficult](https://notso.boringsql.com/posts/deletes-are-difficult/)

---

## Dec 15, 2024

20. **Browser Rendering Process** 
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

## March 09, 2025

21. **NTP and PTP Explained** 
    [How Computers Synchronize Their Clocks - NTP and PTP Explained](https://www.youtube.com/watch?v=WX5E8x3pYqg)

22. **The Obscure System That Syncs All The World’s Clocks** (March 09, 2025)
    [NTP - The Obscure System That Syncs All The World’s Clocks](https://www.youtube.com/watch?v=CwZW0CO7F-g)

23. **NTP vs PTP** [NTP vs PTP](https://www.youtube.com/watch?v=lOUqOEkDT5I)

---

## June 10, 2025

24. DNS resolution 
[Let's Understand DNS For Real](https://www.danielfullstack.com/article/dns-does-not-have-to-be-hard)

---

## July 19, 2025

25. Pirates of the RAG 
[Pirates of the RAG: Adaptively Attacking LLMs to Leak Knowledge Bases](https://arxiv.org/abs/2412.18295)

**"Oops ! I overshared !!!"** Looks like this problem is not unique to humans ! With strategic questioning, RAG based AI spills its secrets !

26. [Refactor of a legacy service to reduce cloud spending](https://blog.duolingo.com/reducing-cloud-spending/) 

I recently completed my 1.5-year streak on Duolingo (just 5 minutes a day—let’s call it dabbling rather than mastery 😅. Yes my Duolingo app icon is on fire !!!)

It feels almost serendipitous to come across this article about how Duolingo slashed cloud costs. One jaw-dropping stat? After the refactor of a legacy service, they uncovered 2.1 billion unnecessary API calls… every single day! 😱

27. [Concurrency Bug but a calm weekend](https://pushtoprod.substack.com/p/netflix-terrifying-concurrency-bug) 

Unconventional technique for a peaceful weekend with a chaos-monkey type self-healing, pragmatic engineering on a Friday afternoon.

How?

- Resizing cluster to meet max capacity
- Randomly kill a few instances every 15 minutes 

Takeaway - Sometimes production incidents can be problem solved to align with quality of life.

28. [HBR: How Generative AI will change Sales](https://hbr.org/2023/03/how-generative-ai-will-change-sales) 

Of all the techno-functional terms I have across this one stands out as remarkably unconventional - "Boundary Spanner".

A “boundary spanner”—an individual who understands and is respected by technical experts as well as by sales team members.
Harvard Business Review

---