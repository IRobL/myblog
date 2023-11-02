---
layout: post
title:  "On Debugging LLM Reasoning via Log of Thought Prompting"
date:   2023-03-03 12:00:00 -0500
categories: AI Prompting neural_networks
---

# On Debugging LLM Reasoning via Log of Thought Prompting
Purusing discussions shared within the LLM community, I bumped into an interesting puzzle the other day.  It's one that a human can do quite easily, but somehow stumps GPT 3.5.  I harbor a lot of intrigue around LLMs, they're probably my new favorite thing to debug.  This post summarizes my thoughts as I investigated this perplexing peculiarity of ChatGPT 3.5.

## Environment
For testing this puzzle, I used OpenAI's playground interface rather than the usual ChatGPT frontend.  This is important for all AI researchers because it allows you to crank the tempurature to 0 which allows for reproducible experiments.  Also, oddly, the first time I feed this puzzle into the WebUI, ChatGPT solved the puzzle correctly, so either I had a lucky forward passes or the model on the frontend is more up-to-date.

###### Settings
Interface: Playground
Model: GPT 3.5-turbo
Temperature: 0
Length: 1535

## Definitions Etc.
Let's touch on definitions and feelings.  Throughout this document, the term reasoning is being used to either:

- refer to any explicit text that expresses the reason a particular conclusion was come, or
- any implicit function whos logic must be approximated in order to come to a particular conclusion

That's similar to how some people will say that planes fly.  Sure it's true that planes are in the air moving, but some might argue the contrary, "no, planes don't fly, they're almost nothing like birds, their 'wings' are rigidly fixed to the body of the plane."  So we can say planes don't fly, submarines don't swim, and neural networks can't reason.  There's something really fascinating about that analogical lineup I'm borrowing from an interview with Linguist Noam Chomsky.  In my mind, planes can fly.  But boats can't swim.  Are submarines barred in the English language from swimming because that's a faculty of humans whereas flight is not and therefore permitted?

Beside the point of that puzzle, the reader may find it more helpful in this context to think of neural networks not so much as "next word predictors" as it is to think of them as **function approximators**.  In the case of these experiments, the LLM truely is approximating the function that one would leverage in articulating the coming to of a conclusion and the reasoning involved.

My new favorite thing to do in Python is to build simple Neural Networks that can be used to approximate mathematical functions.  It's soooo inefficient, but it's been so helpful to see comprehensible algorithms like ECS approximated simply by training a small network of neurons.

Artificial neural networks aren't all sunshine and roses, truely there's something about their existance and capability suite that, in a clever pun, strikes a nerve in the human psyche.  There are two sides to the blade of this strike.  The first is that in some ways, being human is a competition, and naturally anything that begins accumulating human capabilities is going to make an appearance of itself as a contender to our throne.

The second is a boring socia-economic concern almost not worth mentioning given how blatant and obvious the nervous twitch of capitalism is.  Somehow, however, the following paragraph dedicates itself to this expression.  For as long as we can remember, we've lived in a society where our social, emotional, and physiolagical wellbeing has meant doing everything we're idiosynchratically best at to be better at something than as many others as possible.  That fad is probably going to go away in our lifetime and replaced with a new trend that will likely be as or more stable than the current system.  With undeniable apprehention, I'm looking forward to a new system.  While this one has gotten us a long way, there are dark --and vast-- corners of the existing system that are ugly, undignified and vacuously inefficient for what their actual purpose is.  Rather than dive deep into this tangent I'll just say that I'm more comfortable embracing change than I am fearing it.

## The Actual Puzzle
Ok, with definitions out of the way, we can finally get technical.  There aren't a lot of simple questions you can ask ChatGPT 3.5 that completely stump it.  Sure ask it something math-heavy or something about an emergent technology, or something very niche and specialized, and you're sure to be underwelmed by it's uninformed, vapant response, but SOTA LLMs have just about solved simple questions.  Here's a simple one that oddly gives GPT 3.5 a run for it's money (and quite confusingly one solved by open source LLMs).

> Bob is in the living room. He walks to the kitchen, carrying a cup. He puts a ball in the cup and carries the cup to the bedroom. He turns the cup upside down, then walks to the garden. He puts the cup down in the garden, then walks to the garage. Where is the ball? Explain your reasoning.

Here's it's failing output:


> The ball is in the garden.
> <br/><br/>
> Bob initially carries the cup from the kitchen to the bedroom, with the ball inside. Then, he turns the cup upside down in the bedroom. Therefore, the ball falls out of the cup in the bedroom. After that, Bob walks to the garden and puts the empty cup down there. Since the ball is no longer in the cup, it remains in the garden where it fell when Bob turned the cup upside down in the bedroom.

Ok, so the extremely pricy to train GPT 3.5 NN, composed of something like a mixture of 18 expert models of >100b parameters each, has failed to come up with the same answer that any human would come up with.  Why would something that expensive fail?  My first thought was... maybe there's something wrong with the **gating NN** to the MoE and the puzzle is being solved by the wrong expert.  I'm not running GPT 3.5 locally so I have no way of proving that without modifieing the prompt in the hopes of changing the dynamics.  What else could it be?  Maybe there's an **overfitting/ memorization trap** taking place?  Maybe the NN has seen the antics of 'ball' many times before and it knows that 'ball' likes gardens so that's where it is.  Let's come up with one last theory.  Maybe due to the structure of our prompt, the NN has been suggested in some way that ball's location is somehow entangled with Bob's activity despite our human intuitions about local realism.

## Log of Thought Prompting

So CoT and ToT prompting techniques have been discussed this year as ways of squeezing out extra reasoning capabilities from LLMs.  They involve the additional specification to the LLM that they should discuss their thinking.  The intuition I have on this process actually brings us back to next word prediction:

> If you have written a lot of reasoning about something, then predicting an accurate conclusion should be statistically more likely than without any prior reasoning being articulated in the context window.

In this paper, I'd like to present Log of Thought as a piffy name for telling an LLM to log the events that take place in a narrative you've provided, so that it becomes more clear what the LLM is thinking, why(ish), while also eliciting more reasoning material to be added to the context window similar to how CoT and ToT (in part) function.

So for testing our first theory, lets imploy LoT and see what it might be thinking.  I wouldn't be suprised for LoT to solve certain problems outright as that it's fanning the LLM flames the same way CoT does.

###### Log of Thought

```
Bob is in the living room. He walks to the kitchen, carrying a cup. He puts a ball in the cup and carries the cup to the bedroom. He turns the cup upside down, then walks to the garden. He puts the cup down in the garden, then walks to the garage. Where is the ball? Explain your reasoning.

Create a log detailing the status of bob, the cup, and the ball for each event that takes place throughout the above narrative. If Bob hasn't interacted with any of these items, report that their status is unknown until he makes an initial interaction with them. Be sure to account for all forces acting upon the ball (i.e. gravity). Make sure each event clearly states whether the cup is empty or if it has the ball in it. Also, during each event, point out what room the ball is in.  After the log is constructed, please state the final location of the ball.

Example Log Format:

Event 1:
- Bob is in the living room holding a cup.
- status:
  - Bob is in the living room
  - The cup is in the living room
  - The ball's status is unknown
```

###### Failing Response

```

```


###### Prompt: original but make note of gravity???



###### Prompt: original but replace all nouns

TODO: Rewritre, this proves that we aren't in an overtraining failure state where the ball is usually in the garden or some such association.  The lack of a ball also takes us further away from sports commentary which may have pulled us in towards a specific expert that knows something strange about how balls work.  

```
Tim is in the living room. He walks to the kitchen, carrying a crate. He puts a toy in the crate and carries the crate to the bedroom. He turns the crate upside down, then walks to the garden. He puts the crate down in the garden, then walks to the garage. Where is the toy? Explain your reasoning.

Create a log detailing the status of Tim, the crate, and the toy for each event that takes place throughout the above narrative. If Tim hasn't interacted with any of these items, report that their status is unknown until he makes an initial interaction with them. Make sure each event clearly states whether the crate is empty or if it has the toy in it. Also, during each event, point out what room the toy is in. Pay very close attention to the toy as you log events as that the toy cannot be in the crate if the crate is empty. After the log is constructed, please state the final location of the toy.

Example Log Format:

Event 1:

Tim is in the living room holding a crate.
status:
Tim is in the living room
The crate is in the living room
The toy's status and location are unknown
```

###### Response: Failed

Nothing interesting here.  

```
Event 1:

Tim is in the living room holding a crate.
status:
Tim is in the living room
The crate is in the living room
The toy's status and location are unknown

Event 2:

Tim walks to the kitchen, carrying the crate.
status:
Tim is in the kitchen
The crate is in the kitchen
The toy's status and location are unknown

Event 3:

Tim puts a toy in the crate.
status:
Tim is in the kitchen
The crate is in the kitchen with the toy inside
The toy is in the crate

Event 4:

Tim carries the crate to the bedroom.
status:
Tim is in the bedroom
The crate is in the bedroom with the toy inside
The toy is in the crate

Event 5:

Tim turns the crate upside down.
status:
Tim is in the bedroom
The crate is in the bedroom empty
The toy is in the bedroom

Event 6:

Tim walks to the garden and puts the crate down.
status:
Tim is in the garden
The crate is in the garden empty
The toy is in the garden

Event 7:

Tim walks to the garage.
status:
Tim is in the garage
The crate is in the garden empty
The toy is in the garden
```


###### Prompt: 


```
Bob is in the living room. He walks to the kitchen, carrying a cup. He puts a ball in the cup and carries the cup to the bedroom. He turns the cup upside down, then walks to the garden. He puts the cup down in the garden, then walks to the garage. Where is the ball? Explain your reasoning.

Create a log detailing the status of bob, the cup, and the ball for each event that takes place throughout the above narrative. If Bob hasn't interacted with any of these items, report that their status is unknown until he makes an initial interaction with them. Make sure each event clearly states whether the cup is empty or if it has the ball in it. Also, during each event, point out what room the ball is in. Pay very close attention to the ball as you log events as that the ball cannot be in the cup if the cup is empty. Remember all objects obey local realism. After the log is constructed, please state the final location of the ball.

Example Log Format:

Event 1:

Bob is in the living room holding a cup.
status:
Bob is in the living room
The cup is in the living room
The ball's status and location are unknown
```


###### Response: Passed!


```

```
