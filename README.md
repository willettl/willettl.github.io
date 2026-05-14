# My Shit


**Development**
**Medium Scale Abstraction**
**Medium Scale Architecture**
What architectural pattern does your program employ?
Designed the architectural structure for the backend in which we have a folder of models and a folder of services that are intermediate steps for the models. Once we had the layout for the files, then Youssef and I were able to fill in the functions for all the uses of our different tables, and he connected everything to the database and server to make sure we could actually access our tables when we call. All of these things were in separate locations from anything frontend, with only server.js not being in a folder.

What components result from this pattern in your program?
The components that came from this were _____Model.js for each thing, _______Service.js for each thing, db.js and server.js. Each model had at least async function that took in some parameters and would return a row from the table it affects. In each service we built at least one async function that checked for errors and input types then goes and edits the tables using the models. 

What technologies and/or libraries make-up each of the components?
In everything we have it written in javascript, sometimes with bits of SQL queries wrapped up in the functions. The only thing we have explicitly imported into most files are references to each other and db. In server we use express and cors.


**Verification**
**Testing Strategies**
**Testing Infrastructure**

**Design**
What was the design of your user study to evaluate your artifact?
I brought our MVP to a friend who has been into sports betting previously, and asked him to poke around and critique it. In doing this I had to tell him to pretend there were other things going on, such as to imagine a leaderboard, mini-games page and the login buttons.

What were the results of this user study?
The subject said that it all looked good, and that the proposed layout seemed solid. One tiny complaint he had was that even with the other stuff added in there seemed to be a lot of white space on the sides of the bets, whereas other apps he has used feels a lot more full almost to the point of cluttering. Also in other directions just a bunch of white space, which we were trying to model off of official Grinnell websites, seemed to be an issue of his greatly. He also suggested a dark mode, which we are probably going to ignore, but it was an interesting idea nonetheless. The way to interact with the bets made sense for him, although he said it was a little different without an explicit button to press yes or no, but it was easy enough to figure out. 

What were the main points of future development that you took away from this activity?
The main thing I took away is that our proposed layout for functionality and interaction is going to be good. The overall layout of the full website could be optimized to fill space more efficiently, and possibly lead to larger components in the bet, which he said might be a good idea. All in all this evaluation made it seem like we were fully on the right track once we began to add in more functionality. 

**Needfinding**
What techniques did you employ to discover users' needs?
I conducted an interview + observation at the beginning of the process, and then another interview + observation with a different subject after we had our MVP. In the first one, I presented our AI made minigame website and gave her the instructions to just use it however you want. This person was very much a non-gambler, and had no idea how at least half of the games worked. So this process was good to see how someone who has never gambled, either online or in person, would approach being presented with a multitude of options and minimal instructions within the website and games. Then the next part of that was to sit her in front of Polymarket and say “move around like you would, and just tell me if you would bet with fake money (but with leaderboard incentive)”. This again was to see what a fully new person would navigate to, as well as what types of betting habits someone who doesn’t have a crippling gambling addiction would make.  This was the population I was in charge of learning about, because other people were doing this for serious gamblers.

What did you learn about users' needs from this process?
I learned that we didn’t need to make a tutorial or a help section for the website because she fully ignored it on Polymarket and didn’t have a worse experience because of it. It showed that if we had a similar layout, it should be intuitive enough for the average person to be able to figure it out easily. In fact, she was against explanation, as she was fully turned away from playing roulette because there was too much going on, and didn’t really try craps because there was a fair bit of text explanation. While I wouldn’t take this part to heart as much, really making sure to keep our text minimal was emphasized here for sure. The mini-games didn’t have any sort of leaderboard setup, and so she was happy when she won but didn’t care when she lost, which shows promise for roping people into being addicted. But she also said if there was a leaderboard she would have played more carefully, and I assume that sense of caring would translate into the reactions a little bit. When tracking her actions on what she would do with Polymarket, I learned a couple important pieces about betting habits. She instinctively clicked off the home page and right to a category that she felt she knew the most about, which makes sense. Going to what you are most familiar with makes sense, but not looking at the trending page was interesting. This continued as the bets she said she would have made were mostly in the first few categories she looked at, which are the two topics she knew/cared about the most. Was against betting on the nonsense jokey bets because she thought it was stupid, which unfortunately goes against a big piece of what we were trying to get out of this. UI wise, she fully missed all the buttons that were like headers at the top of the page because they were small and out of the way, so we need to make sure we don’t put anything super important there. Also never really looked at the sub-categories on the sidebar, mostly just scrolling the big things in the middle. I think the takeaway from that was that we wanted all the important things to be dead center, and if people are looking for other stuff, then they will find it off to the side.

**Ideation**
What techniques did you use to discover solutions to users' needs?
One big thing we did was find sources we wanted to draw from and build our ideas from there. In the end our goal turned out to be to try and mash together pieces of YikYak and (generic sports betting app) and add Grinnell flare. In talking generally to our friends who use both (not in a strict interview way, but just casually) we found what made each appealing, and tried to pull from there. A large part of our ideation came from the creation of our Timmy Johnny Spike, of which we had four. We laid out what each of our people wanted to get out of the experience of our app and tried to meet all of them, and overlap when possible to minimize the work we had to do. We did a good job of stereotyping Grinnellians and fitting them into molds of caricatures and then almost assigning different pieces of our product to each one.

What solution(s) did you ultimate pursue and why did you choose them?
We felt we needed the leaderboard here, as it overlapped with 2/4 of our Grinnellians (addict, winner) with potential to pull others into more hybrid roles. The comment section felt important to pull in the social user, which we felt would be a large percentage of our user base. Making it as interactive as possible, even if they don’t choose to interact but to watch the conversations that can happen, was a sure way to keep them. This group was more from the YikYak side of things, and I think this is where YikYak gets its big points. Another thing we decided to add to this is a description of a bet in which groups (“Grinnellians” and social lurkers) can be more self expressive, and consume more text.

**Prototyping**
What design questions did you intend to answer with your prototype(s).
One of the big questions we wanted to ask was how intuitive it was to interact with everything, and if in their looking at our MVP, would they be able to identify all the features we plan to have, or would there be something that they would miss, and we need to make it more obvious. A few questions were later asked on the aesthetics of it, but that was not our primary concern and we were sure we were going to change it later (ended up keeping it for the most part in the end).


How did you design your prototype(s) and subsequent user studies to address these questions?
In our MVP/prototype we made sure to include all the areas for things we want to add, even if they had no working actions tied to them yet. Also we chose not to add any instructions or non-essential text to the website, nothing more than the text on the buttons in order to not give any hints or whatever. In the actual interactions I made sure not to give any instruction aside from “this is our gambling website, poke around like you would” in order to see how intuitive it actually was.

What answers to your design questions did your efforts unveil?
It seemed to give us positive results for sure, the subjects all immediately found the bets (obviously) but all tried to click on upvote and comment which did nothing at the time, but they found the pieces of interest on the interactions with the betting. One thing I was worried about was whether or not it was obvious enough that to put money on a bet you had to click on the percentage bar, but thankfully everyone figured that out with no issue, so we were able to keep that design for our final versions of the website. 

**Evaluation**

**Engineering**
**Infrastructure**
What development tools did you use through the process to help write code and automate the build/deployment process?
We used a healthy bit of ChatGPT in our project, and that definitely helped me through some struggles. The way I used it was attempting to write the barebones of my stuff first without help to allow myself to struggle through it first, and then if that fails, give the files to ChatGPT and ask what is going wrong. Because a lot of our files have a similar structure, just doing this on the first of many similar documents, getting help with fixing the first one means I can implement what I learned in the construction of the rest of them. 

What is one specific problem (e.g., debugging and issue) that you encountered while writing code and how did your tools help or hinder your ability to address that problem?
Early on in the process I found issues where the functions to add things and modify the database were running but never saving long term. I kept trying to edit and reexamine the files that the functions were actually in, and even when I threw them into ChatGPT its solutions were not solving it. Eventually I figured out that there was probably something wrong with different files that were causing the issue, and managed to fix it with Youssef super quick. So in this specific case, asking ChatGPT was not super helpful because it was only being fed what I gave it, so it obviously couldn’t detect whatever issue was happening elsewhere.

**Self-learning**
Describe the technology that the tutorial addresses and how it fits into your project.
We did our presentation on SQLite, and while I technically worked with PostGRESQL in the project, the idea of using SQL for our backend was the same. Our project used SQL to store everything from users to markets and bets as a way to make them stay permanent on our website. This is how we kept track of pretty much everything, even seemingly small things such as upvote counts.

Evaluate how useful the tool was in your work. What were its strengths and weaknesses, especially in comparison with other tools you have used.
There is a fair bit of stuff that I created early on with the thought we would use it later on that never really got used, which is just sitting there taking up space and time which was a slight weakness, because it makes sense to set up the schema early on. The strength was that it was great to build functions that called from our front end files easily and filled in things like upvotes and comments smoothly. One thing that was also nice is that in our creation of the tables, we can code in that it creates unique hashes for each element, as well as is able to set default values for fields, which means we didn’t have to put those in other files and make them messier, and could never touch them again.

Identify one particular sticking point to using this technology that you would want your team to know about when adopting the tool and how you resolved it.
I think looking back it would have been easier to modify our schema over time, because when we did have to add new tables it was never super complicated, and it could be tied to preexisting tables without a hitch. This would cut out our wasted space. I also think that we should have had more communication during the early part of our work, because tying in the SQL bits to the front-end that has been built in a bubble so far was tough. To resolve it we just had to go into fields and constants and what not and tie them together which would have been good to put as SQL backend from the start.

**Process**
What software engineering practices did your group employ to manage, distribute, and execute on their work?
We split up early on into  front-end and back-end pairings to distribute both the workload and to cut out the need to learn two different sets of skills, with the idea to come together in the middle of the time and then start piecing the two halves together. We would get updates on work twice a week in meetings Wednesdays and Fridays. We also planned on using Git to push and pull so we could work on our own and all stay up to date.

In your estimation, how well did your group follow through on using these processes throughout the semester?
We split up and stuck with those roles for almost the entirety of the time, and the coming together in the middle and combining stuff did actually go mostly as we planned, if not at a slightly later time than originally imagined. We also all showed up to the meetings to discuss progress. In the end we ended up doing a lot of pushing/pulling while physically together because that is when we had the most work get done at the end, and also when there were more small changes getting sent up to main more often.

What, if anything, got in the way of integrating these process into your workflow?
One thing is that early on, our meetings didn’t seem super productive because there wasn’t a lot of crossover work to be done at that point. We shared updates and future plans, but there wasn’t as much need to be in the same room as there was in the back third of our working time. So maybe we could have structured the load distribution differently to maximise on our in person meetings, or we could have just not needed such long meetings early on in the semester and saved time there. 

**Collaboration**

