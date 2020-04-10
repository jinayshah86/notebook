### Overview of testing categories
**White-box tests** are those that exercise the internals of the code; they
inspect it down to a very fine level of detail. On the other hand, **black
-box tests** are those that consider the software under test as if within a
box, the internals of which are ignored. Even the technology, or the
language used inside the box, is not important for black-box tests. What
they do is plug input into one end of the box and verify the output at the
other end—that's it. There is also an in-between category, called **gray-box
testing**, which involves testing a system in the same way we do with the
**black-box** approach, but having some knowledge about the algorithms and
datastructures used to write the software and only partial access to its
source code.

### In-depth testing categories
- **Frontend tests**: Make sure that the client side of your application is
exposing the information that it should, all the links, the buttons, the
advertising, everything that needs to be shown to the client. It may also
verify that it is possible to walk a certain path through the user interface.
- **Scenario tests**: Make use of stories (or scenarios) that help the tester
 work through a complex problem or test a part of the system. 
- **Integration tests**: Verify the behavior of the various components of your
application when they are working together sending messages through
interfaces.
- **Smoke tests**: Particularly useful when you deploy a new update on your
application. They check whether the most essential, vital parts of your
application are still working as they should and that they are not on fire
. This term comes from when engineers tested circuits by making sure nothing
was smoking.
- **Acceptance tests, or user acceptance testing (UAT)**: What a developer does
with a product owner (for example, in a SCRUM environment) to determine
whether the work that was commissioned was carried out correctly.
- **Functional tests**: Verify the features or functionalities of your
software.
- **Destructive tests**: Take down parts of your system, simulating a failure
, to establish how well the remaining parts of the system perform. These
kinds of tests are performed extensively by companies that need to provide
an extremely reliable service, such as Amazon and Netflix, for example.
- **Performance tests**: Aim to verify how well the system performs under a
specific load of data or traffic so that, for example, engineers can get a
better understanding of the bottlenecks in the system that could bring it
to its knees in a heavy-load situation, or those that prevent scalability.
- **Usability tests, and the closely related user experience (UX) tests**: Aim
to check whether the user interface is simple and easy to understand and
use. They aim to provide input to the designers so that the user experience
is improved.
- **Security and penetration tests**: Aim to verify how well the system is
protected against attacks and intrusions.
- **Unit tests**: Help the developer to write the code in a robust and
consistent way, providing the first line of feedback and defense against
coding mistakes, refactoring mistakes, and so on.
- **Regression tests**: Provide the developer with useful information about a
feature being compromised in the system after an update. Some of the causes
for a system being said to have a regression are an old bug coming back to
life, an existing feature being compromised, or a new issue being introduced.

### Testing guidelines
- **Keep them as simple as possible.** It's okay to violate some good coding
rules, such as hardcoding values or duplicating code. Tests need, first and
foremost, to be as readable as possible and easy to understand. When tests
are hard to read or understand, you can never be confident they are
actually making sure your code is performing correctly.
- **Tests should verify one thing and one thing only.** It's very important
that you keep them short and contained. It's perfectly fine to write
multiple tests to exercise a single object or function. Just make sure that
each test has one and only one purpose.
- **Tests should not make any unnecessary assumption when verifying data.**
- **Tests should exercise the what, rather than the how.** Tests should
focus on checking what a function is supposed to do, rather than how it is
doing it. The type of test you have to write when you concentrate on the
how is more likely to degrade the quality of your testing code base when
you amend your software frequently.
- **Tests should use the minimal set of fixtures needed to do the job.**
- **Tests should run as fast as possible.**
- **Tests should use up the least possible amount of resources.**
- **Anything that crosses the boundaries of your application needs to be
simulated.** We don't want to talk to a real data source, and we don't want
to actually run real functions if they are communicating with anything that
is not contained in our application. A few examples would be a database, a
search service, an external API, and a file in the filesystem.

### Test-driven development (TDD)

**TDD** is a software development methodology that is based on the continuous
repetition of a very short development cycle.

First, the developer writes a test, and makes it run. The test is supposed
to check a feature that is not yet part of the code. Maybe it is a new
feature to be added, or something to be removed or amended. Running the
test will make it fail and, because of this, this phase is called **Red**.

When the test has failed, the developer writes the minimal amount of code
to make it pass. When running the test succeeds, we have the so-called
**Green** phase. In this phase, it is okay to write code that cheats, just to
make the test pass. This technique is called **fake it 'till you make it**. In
a second moment, tests are enriched with different edge cases, and the
cheating code then has to be rewritten with proper logic. Adding other test
cases is called **triangulation**.

The last piece of the cycle is where the developer takes care of both the
code and the tests (in separate times) and refactors them until they are in
the desired state. This last phase is called **Refactor**.

The TDD mantra therefore is **Red-Green-Refactor**.

When you write your code before the tests, you have to take care of **what**
the code has to do and **how** it has to do it, both at the same time. On the
other hand, when you write tests before the code, you can concentrate on
the **what** part alone, while you write them. When you write the code
afterward, you will mostly have to take care of **how** the code has to
implement **what** is required by the tests. This shift in focus allows your
mind to concentrate on the **what** and **how** parts in separate moments
, yielding a brain power boost that will surprise you.

#### Benifits of TDD
- **You will refactor with much more confidence:** Tests will break if you
introduce bugs. Moreover, the architectural refactor will also benefit from
having tests that act as guardians.
- **The code will be more readable:** This is crucial in our time, when coding is
a social activity and every professional developer spends much more time
reading code than writing it.
- **The code will be more loosely coupled and easier to test and maintain:** 
Writing the tests first forces you to think more deeply about code structure.
- **Writing tests first requires you to have a better understanding of the
business requirements:** If your understanding of the requirements is lacking
information, you'll find writing a test extremely challenging and this
situation acts as a sentinel for you.
- **Having everything unit tested means the code will be easier to debug:** 
Moreover, small tests are perfect for providing alternative documentation. 
English can be misleading, but five lines of Python in a simple test are
very hard to misunderstand.
- **Higher speed:** It's faster to write tests and code than it is to write
the code first and then lose time debugging it. If you don't write tests, you
will probably deliver the code sooner, but then you will have to track the
bugs down and solve them (and, rest assured, there will be bugs). The
combined time taken to write the code and then debug it is usually longer
than the time taken to develop the code with TDD, where having tests
running before the code is written, ensuring that the amount of bugs in it
will be much lower than in the other case.
