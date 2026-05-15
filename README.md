# Competencies


## Development
<br>
All code mentioned here can be found in the Grin-gambling/GrinCode repo
<br>

*Medium Scale Abstraction*
<br>
**What functionality does your code sample abstract away?**
It abstracts away the logic of processing a bet, such as making sure there is enough money to make the bet, whether the market is open or not, how we are actually going to run it, and on top of that all the other things that happen alongside placing a bet. We produce a wager yes, but we also need to update the odds, the pool of money in that bet and update the users balance.

**How does it mechanically achieve abstraction?**
In this example we have two different files, where one is in a lower level that has a function get called up to it. The server.js file calls the placeBet function up from services/bettingService.js where a lot of the behind the scenes stuff actually happens:
Even in this file we can see a call to updateBalance, which is a function that lives in an even lower level file in models/userModel.js which is another layer of abstraction. We don’t need to see the exact SQL query it takes to update a users balance, in what table all the information is stored and extra crap when we are looking at placing a bet. We also use updateBalance once again in the file services/marketService.js, which shows how we can use this piece anywhere we want (even though we only had two uses it would be super easy to use this service as much as we wanted).

In server.js (all Youssef):
```javascript
app.post('/api/markets/:marketId/bets', authMiddleware, async (req, res) => {
  try {
    const { marketId } = req.params;
    const { outcomeId, amount } = req.body;

    const wager = await placeBet(req.user.id, marketId, outcomeId, amount);
    res.status(201).json(wager);
  } catch (err) {
    res.status(err.statusCode || 500).json({ error: err.message });
  }
});

```
In services/bettingService.js (both of us wrote together):
```javascript
async function placeBet(userId, marketId, outcomeId, amount) {
  if (Number(user.balance) < numericAmount) {
    throw createError('Not enough acorns to place this bet', 400);
  }

  const wager = await createWager(
    userId,
    outcomeId,
    numericAmount,
    oddsAtBet,
    client
  );

  const updatedUser = await updateBalance(
    userId,
    Number(user.balance) - numericAmount,
    client
  );
}
```
In models.userModels.js (all me):
```javascript
async function updateBalance(userId, newBalance, client = db) {
  const query = `
    UPDATE users
    SET balance = $1
    WHERE id = $2
    RETURNING id, balance
  `;

  const result = await client.query(query, [newBalance, userId]);
  return result.rows[0];
}
```

**What purpose does this abstraction serve in your larger program?**
There are two purposes to this abstraction, the first is to separate the levels for the sake of easier reading and de-cluttering. If we know we have an issue with placing a bet, but the issue is about the money not being taken from an account, we don’t have to look through the entire codebase for anything related to placing a bet, we know it’s something to do with either the model file, or not calling to the model file. The other is that we could use our model functions or service functions wherever we want, and while there wasn’t a significant amount of overlap in the use of these, the structure would have been there had we wanted to.


*Medium Scale Architecture*
<br>
**What architectural pattern does your program employ?**
Designed the architectural structure for the backend in which we have a folder of models and a folder of services that are intermediate steps for the models. Once we had the layout for the files, then Youssef and I were able to fill in the functions for all the uses of our different tables, and he connected everything to the database and server to make sure we could actually access our tables when we call. All of these things were in separate locations from anything frontend, with only server.js not being in a folder. In server.js we also have imported all of our service files to be able to use those as well 

**What components result from this pattern in your program?**
The components that came from this were XXXXXXXXModel.js for each thing, XXXXXXXService.js for each thing, db.js and server.js. Each model had at least async function that took in some parameters and would return a row from the table it affects. In each service we built at least one async function that checked for errors and input types then goes and edits the tables using the models. 
<br>
For example, looking at commentModel.js we have a function createComment (mostly me):
```javascript
async function createComment(marketId, body, client = db) {
  const query = `
    INSERT INTO comments (market_id, body)
    VALUES ($1, $2)
    RETURNING id, market_id, body, created_at
  `;

  const result = await client.query(query, [marketId, body]);
  return result.rows[0];
}
```
This gets called later in commentService (both myself and Youssef):
```javascript
async function addComment(marketId, body) {
  if (!marketId) {
    throw createError('Market ID is required', 400);
  }

  if (!body || !body.trim()) {
    throw createError('Comment body is required', 400);
  }

  const client = await db.connect();

  try {
    await client.query('BEGIN');

    const marketResult = await client.query(
      'SELECT id FROM markets WHERE id = $1',
      [marketId]
    );

    if (!marketResult.rows[0]) {
      throw createError('Market not found', 404);
    }

    const comment = await createComment(marketId, body.trim(), client);
    await client.query('COMMIT');
    return comment;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```
We also see it in server.js which uses addComment from the service (Youssef):
```javascript
app.post('/api/markets/:marketId/comments', async (req, res) => {
  try {
    const comment = await addComment(req.params.marketId, req.body.body);
    res.status(201).json(comment);
  } catch (err) {
    res.status(err.statusCode || 500).json({ error: err.message });
  }
});
```

Then later on we can use app.post functions in our App.tsx file to addComments, like we see here (Front-end folks I believe):
```javascript
const addComment = async (
    marketId: string,
    body: string
  ): Promise<MarketComment> => {
    const response = await fetch(`/api/markets/${marketId}/comments`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ body }),
    });

    if (!response.ok) {
      const errorBody = await response.json().catch(() => null);
      throw new Error(errorBody?.error || "Failed to post comment");
    }

    return response.json();
  };
```

**What technologies and/or libraries make-up each of the components?**
In everything we have it written in javascript, sometimes with bits of SQL queries wrapped up in the functions. The only thing we have explicitly imported into most files are references to each other and db. In server we use express and cors.



## Verification
<br>

*Testing Strategies*
<br>

**Identify a component of your project that you were responsible for testing. According to code coverage tools, what aspects of this component are covered with tests?**
I worked on testing the backend side of things, and we managed to get pretty much all of the backend covered by our testing. All the categories we tested in were: login/logout functions along with session tokens, betting and closing markets, creating and storing comments, checking on bet creation closeouts and timing, and working with up/down votes. These ranges of tests spanned all five of our service.js files, and so we have a corresponding test file for each (found in Grincode in the services folder). We also created a test file for each of the model files in those folders, which were covering the more basic functions to deal with the queries directly (found in Grincode in the models folder). Some of what we did with these are insert/update/select queries, ordering data (transactions, comments),  authenticated market creation, the market lookup queries, and the returned rows shape from each database helper function. 


**What kinds of tests are they?**
We used the vitest unit testing setup that the front-end also used, for even more ease testing together. For the services we utilized a mocked database to run our tests, which we could run different operations for cases with wrong input, for example. For the models they are query unit tests that mock the database client, call the model function, and check the flow of data that the SQL is invoked with the correct parameters and returns the expected row in terms of shape.


**For code that is not covered by tests, why did you not cover them?**
In the backend the only bit we didn’t cover was the schema.sql file and the server.js file. This is because they are on different ends of the spectrum, the schema file is so basic and critical that for any single other thing to work, the schema must be set up in a functional way. We would notice something wrong if everything was breaking at higher levels, but it was all individual issues so it wasn’t anything with schema. And then server.js is so high level that all of its functionality was broken up and tested already, in the services files. So we didn’t need to test it directly because we had already covered testing every piece of it.


*Testing Infrastructure*
<br>

**How have you automated your tests so that they both take "1-click" to run and run automatically as part of build validation?**
In using vitest for our testing environment, we were able to write them all in different files, in a 1-1 ratio of models and services to tests. Then we can configure them in the vite.config.ts file, and we can switch between testing our service files (by using our mock database) or our model files (by using our query based tests). This gets us to a place where we can then run one command in the terminal which grants us a view of every test as well as the total coverage of the files we have tested. This way we can easily see our results in entirety, as well as what bits of code we should think about writing more tests for. 

**Describe a specific occurrence in which your testing infrastructure saved you and/or your team work?**
One thing we were able to find from this, that we never found by clicking around on the website, was issue #50 on github. This issue with this was users were able to create bet markets without being logged into an account. We had clicked around and verified that you couldn’t interact with bets if you weren’t logged in (ie commenting, placing wager, placing a vote), but had never tried to make a market. This issue came from attaching our betPost.tsx to the authentication services, but create bet was a button that lived in app.tsx. While this was an easy fix to implement, not catching it would have been an embarrassing mistake. Had this been a real product launch, users could flood the market with nonsensical bets and override any legitimate attempt to use the product. There would be no consequences, as you couldn’t report a user who isn’t logged in, and so people would have free reign to abuse this hole in the system. Luckily we tested this before “deployment” which saved the work of having to scramble to patch a bug “over the air” and also saved the embarrassment of needing to apologize to our hypothetical customers.


## Design
<br>

*Needfinding*
<br>

**What techniques did you employ to discover users' needs?**
I conducted an interview + observation at the beginning of the process, and then another interview + observation with a different subject after we had our MVP. In the first one, I presented our AI made minigame website and gave her the instructions to just use it however you want. This person was very much a non-gambler, and had no idea how at least half of the games worked. So this process was good to see how someone who has never gambled, either online or in person, would approach being presented with a multitude of options and minimal instructions within the website and games. Then the next part of that was to sit her in front of Polymarket and say “move around like you would, and just tell me if you would bet with fake money (but with leaderboard incentive)”. This again was to see what a fully new person would navigate to, as well as what types of betting habits someone who doesn’t have a crippling gambling addiction would make.  This was the population I was in charge of learning about, because other people were doing this for serious gamblers.

<a href="non-gambler observation.pdf" target="_blank">
  View PDF
</a>

**What did you learn about users' needs from this process?**
I learned that we didn’t need to make a tutorial or a help section for the website because she fully ignored it on Polymarket and didn’t have a worse experience because of it. It showed that if we had a similar layout, it should be intuitive enough for the average person to be able to figure it out easily. In fact, she was against explanation, as she was fully turned away from playing roulette because there was too much going on, and didn’t really try craps because there was a fair bit of text explanation. While I wouldn’t take this part to heart as much, really making sure to keep our text minimal was emphasized here for sure. The mini-games didn’t have any sort of leaderboard setup, and so she was happy when she won but didn’t care when she lost, which shows promise for roping people into being addicted. But she also said if there was a leaderboard she would have played more carefully, and I assume that sense of caring would translate into the reactions a little bit. When tracking her actions on what she would do with Polymarket, I learned a couple important pieces about betting habits. She instinctively clicked off the home page and right to a category that she felt she knew the most about, which makes sense. Going to what you are most familiar with makes sense, but not looking at the trending page was interesting. This continued as the bets she said she would have made were mostly in the first few categories she looked at, which are the two topics she knew/cared about the most. Was against betting on the nonsense jokey bets because she thought it was stupid, which unfortunately goes against a big piece of what we were trying to get out of this. UI wise, she fully missed all the buttons that were like headers at the top of the page because they were small and out of the way, so we need to make sure we don’t put anything super important there. Also never really looked at the sub-categories on the sidebar, mostly just scrolling the big things in the middle. I think the takeaway from that was that we wanted all the important things to be dead center, and if people are looking for other stuff, then they will find it off to the side.

*Ideation*
<br>

**What techniques did you use to discover solutions to users' needs?**
One big thing we did was find sources we wanted to draw from and build our ideas from there. In the end our goal turned out to be to try and mash together pieces of YikYak and (generic sports betting app) and add Grinnell flare. In talking generally to our friends who use both (not in a strict interview way, but just casually) we found what made each appealing, and tried to pull from there. A large part of our ideation came from the creation of our "Timmy Johnny and Spike", of which we had four. We laid out what each of our people wanted to get out of the experience of our app and tried to meet all of them, and overlap when possible to minimize the work we had to do. We did a good job of stereotyping Grinnellians and fitting them into molds of caricatures and then almost assigning different pieces of our product to each one.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f16f22ed-d48b-4a7c-8ad4-8ddec9f013e1" width="45%" />
  <img src="https://github.com/user-attachments/assets/bde02a8b-d672-4f05-9612-259d71716d35" width="45%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/819c2ea9-a394-4b10-9b12-dec3ff153cd5" width="45%" />
  <img src="https://github.com/user-attachments/assets/59ceb58d-c41c-43c3-b0ac-da196af86bf6" width="45%" />
</p>


**What solution(s) did you ultimate pursue and why did you choose them?**
We felt we needed the leaderboard here, as it overlapped with 2/4 of our Grinnellians (addict, winner) with potential to pull others into more hybrid roles. The comment section felt important to pull in the social user, which we felt would be a large percentage of our user base. Making it as interactive as possible, even if they don’t choose to interact but to watch the conversations that can happen, was a sure way to keep them. This group was more from the YikYak side of things, and I think this is where YikYak gets its big points. Another thing we decided to add to this is a description of a bet in which groups (“Grinnellians” and social lurkers) can be more self expressive, and consume more text.

*Prototyping*
<br>
**What design questions did you intend to answer with your prototype(s).**
One of the big questions we wanted to ask was how intuitive it was to interact with everything, and if in their looking at our MVP, would they be able to identify all the features we plan to have, or would there be something that they would miss, and we need to make it more obvious. A few questions were later asked on the aesthetics of it, but that was not our primary concern and we were sure we were going to change it later (ended up keeping it for the most part in the end).


**How did you design your prototype(s) and subsequent user studies to address these questions?**
In our MVP/prototype we made sure to include all the areas for things we want to add, even if they had no working actions tied to them yet. Also we chose not to add any instructions or non-essential text to the website, nothing more than the text on the buttons in order to not give any hints or whatever. In the actual interactions I made sure not to give any instruction aside from “this is our gambling website, poke around like you would” in order to see how intuitive it actually was.

**What answers to your design questions did your efforts unveil?**
It seemed to give us positive results for sure, the subjects all immediately found the bets (obviously) but all tried to click on upvote and comment which did nothing at the time, but they found the pieces of interest on the interactions with the betting. One thing I was worried about was whether or not it was obvious enough that to put money on a bet you had to click on the percentage bar, but thankfully everyone figured that out with no issue, so we were able to keep that design for our final versions of the website. 

*Evaluation*
<br>

**What was the design of your user study to evaluate your artifact?**
I brought our MVP to a friend who has been into sports betting previously, and asked him to poke around and critique it. In doing this I had to tell him to pretend there were other things going on, such as to imagine a leaderboard, mini-games page and the login buttons.

**What were the results of this user study?**
The subject said that it all looked good, and that the proposed layout seemed solid. One tiny complaint he had was that even with the other stuff added in there seemed to be a lot of white space on the sides of the bets, whereas other apps he has used feels a lot more full almost to the point of cluttering. Also in other directions just a bunch of white space, which we were trying to model off of official Grinnell websites, seemed to be an issue of his greatly. He also suggested a dark mode, which we are probably going to ignore, but it was an interesting idea nonetheless. The way to interact with the bets made sense for him, although he said it was a little different without an explicit button to press yes or no, but it was easy enough to figure out. 

**What were the main points of future development that you took away from this activity?**
The main thing I took away is that our proposed layout for functionality and interaction is going to be good. The overall layout of the full website could be optimized to fill space more efficiently, and possibly lead to larger components in the bet, which he said might be a good idea. All in all this evaluation made it seem like we were fully on the right track once we began to add in more functionality. 



## Engineering
<br>

*Infrastructure*
<br>

**What development tools did you use through the process to help write code and automate the build/deployment process?**
We used a healthy bit of ChatGPT in our project, and that definitely helped me through some struggles. The way I used it was attempting to write the barebones of my stuff first without help to allow myself to struggle through it first, and then if that fails, give the files to ChatGPT and ask what is going wrong. Because a lot of our files have a similar structure, just doing this on the first of many similar documents, getting help with fixing the first one means I can implement what I learned in the construction of the rest of them. We also used Github to collaborate in our codebase, which was useful at some times and harmful at others. The issues tab was nice in the beginning to plan out what we wanted to get done and at levels of priorities. In actuality we ended up doing the bulk of our work together, so we never used merge conflicts in the traditional way as much. This did kick us once or twice because we had a few instances of overriding by not pulling and pushing or something of that sort. I also chose to use VSCode as my way to work on editing code. I don't have a huge setup or anything, mainly just the extensions that we directly needed and I wasn't using Copilot so it wasn't like this was an amazing tool, but it was solid enough for the work I wanted to do and not overly complicated to where it was another thing to juggle.

**What is one specific problem (e.g., debugging and issue) that you encountered while writing code and how did your tools help or hinder your ability to address that problem?**
Early on in the process I found issues where the functions to add things and modify the database were running but never saving long term. I kept trying to edit and reexamine the files that the functions were actually in, and even when I threw them into ChatGPT its solutions were not solving it. Eventually I figured out that there was probably something wrong with different files that were causing the issue, and managed to fix it with Youssef super quick. So in this specific case, asking ChatGPT was not super helpful because it was only being fed what I gave it, so it obviously couldn’t detect whatever issue was happening elsewhere.

*Self-learning*
<br>

**Describe the technology that the tutorial addresses and how it fits into your project.**
We did our presentation on SQLite, and while I technically worked with PostGRESQL in the project, the idea of using SQL for our backend was the same. Our project used SQL to store everything from users to markets and bets as a way to make them stay permanent on our website. This is how we kept track of pretty much everything, even seemingly small things such as upvote counts.

**Evaluate how useful the tool was in your work. What were its strengths and weaknesses, especially in comparison with other tools you have used.**
There is a fair bit of stuff that I created early on with the thought we would use it later on that never really got used, which is just sitting there taking up space and time which was a slight weakness, because it makes sense to set up the schema early on. The strength was that it was great to build functions that called from our front end files easily and filled in things like upvotes and comments smoothly. One thing that was also nice is that in our creation of the tables, we can code in that it creates unique hashes for each element, as well as is able to set default values for fields, which means we didn’t have to put those in other files and make them messier, and could never touch them again.

**Identify one particular sticking point to using this technology that you would want your team to know about when adopting the tool and how you resolved it.**
I think looking back it would have been easier to modify our schema over time, because when we did have to add new tables it was never super complicated, and it could be tied to preexisting tables without a hitch. This would cut out our wasted space. I also think that we should have had more communication during the early part of our work, because tying in the SQL bits to the front-end that has been built in a bubble so far was tough. To resolve it we just had to go into fields and constants and what not and tie them together which would have been good to put as SQL backend from the start.

*Process*
<br>

**What software engineering practices did your group employ to manage, distribute, and execute on their work?**
We split up early on into  front-end and back-end pairings to distribute both the workload and to cut out the need to learn two different sets of skills, with the idea to come together in the middle of the time and then start piecing the two halves together. We would get updates on work twice a week in meetings Wednesdays and Fridays. We also planned on using Git to push and pull so we could work on our own and all stay up to date.

**In your estimation, how well did your group follow through on using these processes throughout the semester?**
We split up and stuck with those roles for almost the entirety of the time, and the coming together in the middle and combining stuff did actually go mostly as we planned, if not at a slightly later time than originally imagined. We also all showed up to the meetings to discuss progress. In the end we ended up doing a lot of pushing/pulling while physically together because that is when we had the most work get done at the end, and also when there were more small changes getting sent up to main more often.

**What, if anything, got in the way of integrating these process into your workflow?**
One thing is that early on, our meetings didn’t seem super productive because there wasn’t a lot of crossover work to be done at that point. We shared updates and future plans, but there wasn’t as much need to be in the same room as there was in the back third of our working time. So maybe we could have structured the load distribution differently to maximise on our in person meetings, or we could have just not needed such long meetings early on in the semester and saved time there. 

*Collaboration*
<br>

**Identify four different instances where you participated in collaborative work with your team as identified by specific commits, pull requests, and/or issue numbers on Github. These instances should not simply be "committing code" but situations that required back and forth with your peers, e.g., a substantial bug or a code review. You should have one instance each of (1) filing and/or resolving a bug, (2) filing a pull request, and (3) performing a code review of a pull request.**
Reviewing pull request #23
Working on fixing bug #6
Producing pull requests #48 and #56
Working on comments storing, which was mostly in person but collaborative

**Identify the work you did in each of these instances.**
For working on bug #6, this was early on in the project when our backend wasn’t entirely setup yet and the frontend had a button that was ready to start creating bets. At this point I had setup the framework in models and services that held most of the work to start adding things to a database. But I needed help to connect it to what the front-end people had done already. Youssef was a big help here, and took what I had built already and added in the server bit that was needed for the final connection, and then we both worked with either Sam or Mina (I forget) to be able to see where in the code we needed to attach our half.
During the accessibility, before I produced pull requests #48 and #56, I was working on a file I had touched before, but not one that I had built by myself, and one that had been through a lot of changes recently. Especially during the beginning I needed a lot of help from the front-end people to point me to how things worked, syntax and format of the file. At the very end when I was pushing to main, I needed them to look at my pull requests to make sure I didn’t accidentally break anything, because my only indicator for correctness was no red squiggly lines. So having them double check my work to make sure I only did what I wanted was a good piece of collaboration.
When reviewing a pull request #23, I had to make sure that there were no major merges that would mess up something. It mostly entailed going through code that she had previously written and then deleted which made my job fairly easy. There were a few pieces where it seemed to be a genuine decision to make, but knowing what the goal of the pull request was and having an understanding enough of the previous code made the decisions easy to make. I also made sure to comment on the next steps that lead from this pull request, because it wasn’t exactly the closing of an issue but a step to our final product here.
When we were working on comments and upvotes storing, we knocked it out all in one sitting as a group so there was no back and forth on Github but we needed a lot of communication. Youssef did lots of the writing for the services while I was trying to get tables and basic functions set up. I also was talking to Mina and Sam trying to figure out how they had done the comments and upvotes getting them to temporarily stay and not permanent.

**Describe the difficulties that necessitated communication with your peers in each instance.**
For bug #6 the difficulty was that I hadn’t reached the point in my self learning where I knew how to start bridging the gap between server and front-end yet, and Youssef already had that knowledge. From that point we figured it would help to have someone who knew how the front-end was working in order to click them together smoothly.
In pull requests #48 and #56, I didn’t have many issues with communicating. The issues I faced were purely with dealing with the accessibility framework itself, but to be fair we were in person during this. I think if I had to ask questions to people over whatsapp or email or whatever I would have a very different answer, but because I was sitting next to Sam I was easily able to get constant knowledge on the code I was working with.
In the couple merge decisions that actually conflicted I had to go back and read through the original code first to re-remember what it was actually doing, because this was front-end and I wasn’t as familiar with it. So that was a little annoying but in the end the thorough check-over made it for a higher quality pull request review than if I had just skimmed over it. This wasn’t exactly an issue in communication as it was with me, but I guess it could have been Sam to merge this branch and not me.
In communicating that I needed to find where to attach the backend from Youssef, and Mina and Sam pointing to where in the code base it got called, I got confused and it took a while to find another thing elsewhere in the file that needed to be changed as well. Between not knowing exactly what Youssef was doing and not being entirely knowledgeable on how the main file was run, I missed a const and something in post props that needed to be changed for a while, drawing out the process.

**Did you encounter any difficulties in collaboration with team during these instances? If so, how did you resolve those difficulties?**
Not too many difficulties working on these specific issues with the team, I think the only thing I can think of was that when we worked remotely, which was rare for doing big parts of the project, slow communication killed momentum sometimes as I would get a response too late to where I was unmotivated and locked out of doing the project. This wasn’t too big of an issue because we did most of our big work in person together. When working together somewhat of an issue was assuming base knowledge between front and back end, which left us glossing over what we assumed was trivial information.


## Advanced
<br>

*Accessibility*
<br>

**Identify a portion of the user interface that you designed.**
How does your design adhere to one or more of the principles of accessible design?
The design I added was to help with screen readers for accessibility, working on the principle of robustness. The main thing to fix was that our tags were for the longest time all <div> which are not helpful. I worked on the file app.tsx and button.tsx minorly along with Mina, which had a few types of issues to fix. The easy one was to add labels onto various things, such as buttons. So we now have aria-label on buttons at the top of the page that lead to minigams, login etc. We also added sections to the app and labeled those, so the leaderboard got a label as well as betting markets and top bets. Then we added some structure to the file, giving the whole thing a labeled wrapping, and then giving the banner a better label. We added screen reader announcements for loading/errors which we never really saw when running but felt good on principle. Another thing was adding in a bit that said which page we were on, either the market or the mini game. Independently I also messed around in the button.tsx file to add a better outline function, which allowed it to outline every single button without fail, because for some reason it was not going on the upvotes and report buttons.


**How did you use validation tools (if at all) to ensure that your design was accessible?**
We used the WAVE extension to get a look at our scores. We got a 7.9/10 and it seemed like most of the points being docked were for low contrast pieces of text, which we deemed as pretty unnecessary pieces of text and so didn’t worry. But this helped us track the structure of our file properly to make sure the headers and layouts worked. Then we also got to see the order, which we used to make sure that all of the buttons we wanted to be known as buttons worked, and the order that they would tab around in.


