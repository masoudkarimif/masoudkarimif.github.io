# Recent Reads

  <a target="_blank" rel="nofollow" href="https://www.goodreads.com/user/show/26483728-masoud"><img alt="goodreads.com" style="border:0" src="https://s.gr-assets.com/images/widget/widget_logo.gif" /></a>

<br>

---


### Understanding Software

<p align="center" style="float: left; margin: 0 20px 10px 0;">
  <img src="https://images-na.ssl-images-amazon.com/images/S/compressed.photo.goodreads.com/books/1507678200i/36389464.jpg" alt="Understanding Software" width="200"/>
</p>

This book contains a series of short essays on everything related to Software Engineering. Compared to Max's other book, Code Simplicity, this one is bigger, more comprehensive, and hence, covers more topics.

If I had to pick only one thing from the book, it'd be: the difference between an average and a rockstar programmer is understanding. "All you have to do to become an excellent programmer is fully understand what you're doing." It may seem obvious and non-revolutionary at first sight, but speaking from experience, it is as important as it's true. As the author puts it, "The vast majority (90% or more) of programmers have absolutely no idea what they are doing." Therefore, the code they write is hard to read, maintain, and debug. Examples of violating this principle that I've seen include but are certainly not limited to: not using joins when working with relational databases and performing the joins in the code (not understanding relational databases); using asynchronous programming in CPU-intensive applications to boost performance (not understanding async programming and when to use it); and using Python threads to achieve parallel computing (not understanding the Python GIL).

Max loves simplicity not only when it comes to writing software, but also writing books. I've also had the pleasure of talking to him a few times as he's an advisor to our company. He certainly knows what he's talking about.

---

### Code Simplicity: The Fundamentals of Software 


<p align="center" style="float: left; margin: 0 20px 10px 0;">
  <img src="https://m.media-amazon.com/images/I/514tZi6ZbsL._SX375_BO1,204,203,200_.jpg" alt="Code Simplicity" width="200"/>
</p>

This is a short book on—as the title suggests—writing simple code, as the root cause of every problem in software is complexity.

The main purpose of any software, the author believes, is to help people. Therefore, your ability to write good code is determined by your ability to imagine ways to assist others. Speaking from experience, that makes sense. Every developer I’ve seen writing bad code—and by bad I mean code that can’t be understood easily by others and is hard to be changed in the future—has written code for themselves, not for others. Some even put extra effort to make their code less readable as they think—wrongly, of course—that it would make them irreplaceable at their company. So, if you’re an engineering manager and find a developer who writes bad code and never makes an effort to improve, you need to make a decision sooner rather than later.

Max also argues that “It is more important to reduce the effort of maintenance than it is to reduce the effort of implementation.” The reason is that most software systems are maintained for a long time, therefore the effort of implementation becomes less and less significant; the effort of maintenance, on the other hand, is what really matters. So, write code that’s easy to maintain even if it takes you more time to write it. How? Follow software development best practices such as DRY and SOLID, and most importantly, make your code as simple as possible; or as Max puts it, “dumb simple.”

All in all, this is a great book. However, if you're looking for code samples, you're not going to find that many.

---

### A Philosophy of Software Design


<p align="center" style="float: left; margin: 0 20px 10px 0;">
  <img src="https://images-na.ssl-images-amazon.com/images/S/compressed.photo.goodreads.com/books/1531857377i/39996759.jpg" alt="A Philosophy of Software Design" width="200"/>
</p>

I disagree with the author when it comes to comments in your code. Not only Ousterhout does believe in having comments in almost everywhere in your code--interfaces, implementations, configs--he goes so far as to suggest a Comment Driven Design (CDD)--although that's the name I gave to his principle; it's not what he calls it.

In his view, it's a good practice to start designing a program by writing comments about what different parts should do, and then implementing them. Comments, as
Max Kanat-Alexander puts it, should only describe "why" we're doing what we're doing, and not "what" we're doing; that should be clear from the code itself. If it's not, improve your code and make it more readable. I'm with Max on this.

The book also focuses on OOP, so you'll enjoy this book more than I did if you like that paradigm.