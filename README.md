# Links

1. **8 versions of UUID and when to use them** (August 25, 2024)
   [https://ntietz.com/blog/til-uses-for-the-different-uuid-versions](https://ntietz.com/blog/til-uses-for-the-different-uuid-versions)

   Key takeaway: UUIDs have 8 versions, each with specific use cases. V4 (random) and V7 (sortable) are most common.

---

2. **Reverse Engineering TicketMaster's Rotating Barcodes (SafeTix)** (August 25, 2024)
   [https://conduition.io/coding/ticketmaster](https://conduition.io/coding/ticketmaster)

---

3. **Database Caching** (August 31, 2024)
   [Database Caching](https://www.prisma.io/dataguide/managing-databases/introduction-database-caching)

---

4. **SaaS Pricing Models** (September 13, 2024)
   [SaaS Pricing Models](https://www.cobloom.com/blog/saas-pricing-models#)

---

5. **One Minute Focus** (September 13, 2024)
   [One Minute Focus](https://oneminutefocus.com/)

---

6. **B-trees and database indexes** (Oct 4, 2024)
   [B-trees and database indexes](https://planetscale.com/blog/btrees-and-database-indexes)

---

7. **B+ tree visualiztion** (Oct 4, 2024)
   [B+ tree visualiztion](https://bplustree.app/)

---

8. **B tree visualiztion** (Oct 4, 2024)
   [B tree visualiztion](https://btree.app/)

---

9. **Bitten by Unicode** (Oct 4, 2024)
   [Bitten by Unicode](https://pyatl.dev/2024/09/01/bitten-by-unicode/)

---

10. **Neuroscientist: How to LEARN ANYTHING Without Any Effort** (Oct 4, 2024)
    [Youtube - Some strategies for learning more effectively](https://www.youtube.com/watch?v=I2dm72OuK6M)

**Key takeaways**

- Pause for 10 seconds
- Focus and Rest
- Every 90 min have 10 breaks of 20-30 seconds break, randomly (this is the key)

---

11. **45 ways to break an API server** (Nov 16, 2024)
    [45 ways to break an API server(negative tests with examples)](https://dev.to/zvone187/45-ways-to-break-an-api-server-negative-tests-with-examples-4ok3)

---

12. **Cuckoo hashing Visualizations** (Nov 16, 2024)

- [Interactive Cuckoo](https://itu.dk/people/maau/teaching/visualisation/cuckoo-hashing/interactive.html)

- [Cuckoo Hashing](https://itu.dk/people/maau/teaching/visualisation/cuckoo-hashing/index.html)

- [Another Cuckoo Hashing visualization](https://www.lkozma.net/cuckoo_hashing_visualization/)

---

13. **Claude Artifacts** (Nov 16, 2024)

- [Claude Artifacts](https://simonwillison.net/2024/Oct/21/claude-artifacts)

- [Claude transcript for Extracting links from pasted HTML](https://gist.github.com/simonw/0a7d0ddeb0fdd63a844669475778ca06)

---

14. **Accountability sinks** (Nov 16, 2024)
    [Nobody is responsible](https://aworkinglibrary.com/writing/accountability-sinks)

---

15. **Understanding Round Robin DNS** (Nov 16, 2024)
    [Round robin DNS visualization](https://blog.hyperknot.com/p/understanding-round-robin-dns)

---

16. **Art of attention** (Nov 16, 2024)
    [Concentration](https://billwear.github.io/art-of-attention.html)

---

17. **Binary vector embeddings and Matryoshka embeddings** (Dec 02, 2024)
    [Binary vector embeddings are so cool](https://emschwartz.me/binary-vector-embeddings-are-so-cool)

---

18. **Prioritizing sanity** (Dec 02, 2024)
    [How We Built a Self-Healing System to Survive a Terrifying Concurrency Bug At Netflix](https://pushtoprod.substack.com/p/netflix-terrifying-concurrency-bug)

---

19. **Deletes are difficult** (Dec 10, 2024)
    [Deletes are difficult](https://notso.boringsql.com/posts/deletes-are-difficult/)

---

20. **Browser Rendering Process** (Dec 15, 2024)
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
