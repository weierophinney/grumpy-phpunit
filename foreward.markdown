{frontmatter}
# Foreward 
_Justin Searls is what I like to call a "testing brother from another mother".
He works with the awesome folks at [Test Double](http://testdouble.com) providing
consulting services for people doing Ruby and Javascript, with a focus on
testing. He and I have had some interesting discussions about testing in general
and I asked him to provide his perspective on the benefits of testing._


I first used PHP to build a web application in 2004. By that time, I'd already been taught -- and had failed to learn the benefits of -- unit testing and even test-driven development. In this case, I just wanted to get my application written, with as few secondary concerns to worry about as possible. 

Developing in PHP made me happy. Both the language and its community seemed focused squarely on helping me get things done, and they didn't impose an expectation that I adopt any highfalutin' practices or conventions. As a result, I was iterating on a working application from day one.

And "day one" was hugely productive! The same went for "week one". As it happened, "month one" was such a success that I had completed the minimum requirements of an application for which I'd been given over 20 weeks' worth of budget.

I soon discovered, however, that there was trouble in paradise. Slowly, adding small features had begun to take me significantly more time than I'd been accustomed to large features taking. While I'd previously been a beacon of workplace optimism and excitement, I noticed that I was spending a larger proportion of my day feeling confused and frustrated. I wasn't only dreading showing up for work, I'd even begun to resent my editor's application icon, because clicking it meant that I was in for a bad time.

The root cause, and it's plainly obvious to anyone who might read my code, is that the source had grown into a tangled jungle of rogue mega-functions, replete with trees of towering switch statements and swamps of nested if-else constructs. The code lacked any rhyme or reason. It contained no helpful signposts to my future self. Concerning myself with the design of the code hadn't seemed necessary, because my intimate, almost instinctual knowledge of the code had made me so productive at the project's outset.

But at some point, the complexity of my application's code had increased beyond what I could hold in my brain at a single time. Prior to that point, I had been unfettered; liberated from formalities like up-front design or maintaining automated tests. Beyond that point, though, I could sense that my haste had cornered me into a dead end. The only question was: could I re-bottle whatever genie I'd released to help me become so productive in the first place?

In the case of that application, the answer was "no"; I couldn't make things better. The drastic internal changes necessary to make the application easier to maintain would have required confidence that my changes didn't break anything. At the time, I lacked the expertise to write the automated tests I would have needed to gain said confidence to make any major changes. Ultimately, I relented and stayed on the path of most resistance, muscling my way through to the end of the project. Needless to say, no one was as impressed with my productivity during the final leg of the project as they had been after its glorious "month one".

At this point, it might seem like I told this cautionary tale for the purpose of exhorting to readers, "*see?! That's why you start writing tests for everything on day one! Because even if you feel great today, someday your application will explode with complexity and you'll be miserable*". I won't say that, however, because that would be a specious, dogmatic argument and it would oversimplify the world of rich and subtle challenges facing software developers. There are, after all, many ways to ship working software with acceptable maintainability; excellent software can absolutely be written without any automated tests at all.

So if testing isn't absolutely necessary for success, why is it valuable? The answer is that automated tests are an excellent tool for establishing tight, rapid feedback loops when creating and changing code. To explain, let me rewind to the beginning of my story. 

I said that working with PHP made me happy, because I felt the extraordinary positive feedback of seeing something, albeit small, start working in the first couple of hours. Not only that, but I could get ongoing feedback that my changes worked as quickly as I could save a file and refresh a browser window. It was only once my application had grown significantly that the feedback loop had started to slow down -- a five second pause here, a few clicks to set up the app there, perhaps a few minutes to verify the change didn't introduce any bugs. What I eventually realized that I wanted was for every day to feel like "day one" of a fresh project; I wanted a rapid feedback loop to be sustainable for the life of the application.

In the past, I had tried building applications of a similar scope with other tech stacks, like Java, known for their long-term "hardiness". But my projects never seemed to get off the ground. I failed in part because I'd sink the first, crucial hours of my motivation into troubleshooting while setting up the recommended build tools and supporting libraries. And even when I managed to clear that hurdle, any sense of progress was stymied by the nagging doubt that my architecture would elicit the judgment of my contemporaries. Perhaps a more durable technology stack or application design would have made my rapid feedback loop more sustainable in the long run, but the initial "short run" was so painful that I'd never find out what the long run" felt like.

It turns out that a "sense of progress" is crucial to productive software development. Feedback, both positive ("that worked!") and negative ("that doesn't!") has a huge impact on the psyche of the developer. When people endeavor to build tangible things, they receive concrete feedback about progress constantly (e.g. "the table has two legs, so I'm about halfway done"; "this book introduction has 283 words, so I've got a ways to go"). But when building something as ephemeral as software, progress comes in fits and starts, sometimes to the point of feeling illusory. 

The magic of unit testing, particularly in the context of test-driven development, is that it gives the developer the ability to control his or her own sense of progress. Traditional feedback requires our fully integrated application and our eyeballs to assert whether we're on the right track or wrong track. Unit testing allows us to establish a feedback loop at whatever level-of-integration we wish (e.g. perhaps a bunch of objects in coordination, perhaps one function in pure isolation), so long as we can imagine and implement a way to assert working behavior that doesn't require our eyeballs' manual inspection. In this way, even if we're faced with implementing a solution of daunting complexity, unit tests can usually help us break the problem down in such a way that we can make (and importantly, feel!) incremental progress on our path to overall completion.

By mastering both the tools and craft of unit testing, rapid feedback is attainable regardless of the age or complexity of the project. At the outset of an application's life, a failing test can help us set up the critical infrastructure of our application, and we can get some motivating feedback even if there's nothing visible to users yet. And for a mature project, however tangled its source, a test can usually be crafted to gain certainty that a change was made safely and successfully.

When applied rigorously and consistently, and when the pain of a hard-to-write test is responded to with improvement to the design of the code being tested, developers can hope to remain happy and productive over the life of any application. 

There's a lot more to learn about how to get there, but that's what the rest of this book is for! My hope is that the lessons that Chris has prepared in this book will serve to equip you with the skills to someday find yourself working on a mature application while feeling as productive as you did on day one.

- Justin Searls
