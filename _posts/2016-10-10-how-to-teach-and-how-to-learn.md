---
title: "How to teach X to people who've never learned X"
layout: post
---

There has been a lot of efforts in the Rust community to explore ways to teach Rust to people who've never programmed. This prompted me to think about learning and teaching, and a lot of this will be my personal stories. So get on board my train of thoughts! 🚂 <small>If you want to skip to the Rust part, click <a href="#how_to_teach_rust">here</a>.

## <a href="#challenges_as_a_learner" id="challenges_as_a_learner">Challenges as a Learner</a>
I first learned how to program about 3 years ago. I was a biology researcher at a biotech startup, making yeast strains that create spider silk. I was also living in San Francisco at the time, where programming was *everywhere*. So I decided to give it a try completely out of city-wide peer pressure. Someone told me I should start from "Hello World," and after a week or so, I stopped.

Why did I stop learning? What discouraged me from going further?

To me at that time, making computers print things (which is usually the first step in many tutorials) was not inherently exciting. I knew nothing about computers at that time, and I remember thinking *what's so fun about making letters appear on the screen?* This seems to be a common first roadblock to many people. Ashley Nelson-Hornstein during her [keynote talk at Strange Loop](https://youtu.be/fNe1i7nVbXI?t=47m53s) talked about how programming a calculator was not an exciting way to learn programming for her, because she already owned one. I didn't find printing "Hello World" on my computer fun, because I could already make a slideshow where the words spin and change colors! I can already do that without knowing how to program, so why should I bother learning how to make computer print boring looking letters? It was difficult for me to see the application.

This brings my train of thoughts to going to [Portland State Aerospace Society](http://psas.pdx.edu/) meeting for the first time recently. A super friendly mechanical engineer showed us the rocket room, where the magic happens. He talked about the different kinds of fuels (things that explode!), a rocket engine nozzle made by 3D printing metal (I didn't even know that was possible!), and the sad accidents that happen when the parachute doesn't deploy correctly (crashing rockets!). It was no doubt one of the coolest things I've seen lately.

Then a software engineer showed us the fruit of his labor -- a computer that prints its orientation perceived by a gyroscope. This information is very important for the rocket to correct its trajectory when it's flying in the sky. That is really, **really** cool. Think about all the sensors that interact with each other, and how the computer parses that information to a human readable format! What about delivering that data to other rocket parts so that it can correct its orientation? Plus, the computer could communicate with the gyroscope over wifi!

But the thing is, me 3 years ago wouldn't have found this computer as interesting as the other parts of the rocket. The computer just prints numbers after all. Me at present day find it so fascinating that I want to get my hands messy with it. And it's probably because I could imagine the challenges behind this, and I have some understanding of how computer works. If you want to appreciate something, you have to know some basics of it. And the road to knowing the said basics can be quite mundane.

## <a href="#challenges_as_a_teacher" id="challenges_as_a_teacher">Challenges as a Teacher</a>

The challenge in teaching is keeping learners motivated and interested during that initial hump of learning, so they can have a blast later. I recently had to think about this hard when I was a Teaching Assistant for the intro to biology course for undergraduates.

When I tell people I studied biology, people sometimes say "I tried studying biology, but I stopped because it was too much memorization." This makes me feel sad because they were discouraged by that first hump. To picture why, this would be equivalent to saying "I tried studying programming, but it was too much memorizing syntax." Yes, remembering syntax is important, but that's the first step to making and understanding a system that's much larger and more fun.

My goal as a Biology TA was to make this first step of building foundation as painless as possible so my students will stay interested. After that, hopefully they could see what I was seeing, which is a much larger system called biology. Every living organism is a dynamic, intricate system that has a specification scripted in DNA. When you have learned the foundation (or "memorized" it), you *understand* why bread tastes sweet when you chew for a long time<sup><a href="#ref_0" id="0">0</a></sup> or why not finishing that antibiotics prescription is so dangerous<sup><a href="#ref_1" id="1">1</a></sup>.

So what attempts did I make?

1. I tried so hard<sup><a href="#ref_2" id="2">2</a></sup> to keep the scope of learning small. Beginners don't have to know *everything*, but just the necessities to understand the current scope. Then we expand the scope by a bit. Should we talk about the hydrogen bonds that drive adenine to bind with thymine, when this is the first time we are hearing the word DNA? Probably not. Will it be really cool to dive deep later? YES. Similarly, is it necessary to learn that there are multiple kinds of garbage collectors, when we don't know anything about garbage collection? No. Will it be cool to learn them later when we have some knowledge of garbage collection? Absolutely yes.

2. I tried to connect the new concepts to something they are already familiar with. When teaching about lactic acid fermentation, we could start by explaining the stages in carbohydrates to ATP conversion and the role of oxygen in this process. But maybe the better way is to start from what people already know by asking "What is the common thing among kimchi, yogurt, and muscle soreness?"<sup><a href="#ref_3" id="3">3</a></sup> Similarly, can we explain the concept of computer's garbage collection by making an analogy to a city's garbage collection system? Imagine a city without any garbage collection, and how stinky it would be because of the overflowing garbage. You'll be stuck because of the garbage taking up space everywhere! This also makes it easier to visualize a perfect city that doesn't need a garbage collection, because it doesn't even create *any* garbage! 😉

## <a href="#motivation_to_learn" id="motivation_to_learn">Motivation to Learn</a>

### Having a Clear Goal
I wrote earlier that I first learned how to program 3 years ago and stopped after a week. What brought me back to pick up where I left off a little more than a year ago? This is the interesting and important part.

One day, I wanted to make a spreadsheet where one column was a gene symbol, and the next column was its corresponding protein's function. I thought of going through online databases and manually copy and pasting them. Then my coworker told me that I could write a program to do it automatically. I said "YOU CAN DO THAT???" And then I thought *I want to learn how to do this*. I found a goal that I wanted to reach, and learning how to print "Hello World" was suddenly not so bad.

I find myself saying the same thing when I get excited to learn something new. You can make a hat out of a skein of yarn?? *How? Show me!* You can learn how to make a mug from clay? *I want to learn*. You can make a rocket as an amateur? *I want to learn!!* I think having a clear goal helps me stay motivated through the first phase of learning.

### Just Having Fun
The motivation to learn might come from just having fun too. I like ceramics because it's fun to see the glaze change colors when fired and transform a ball of clay into a vase. Next thing I realize, I'm learning about inorganic chemistry and centrifugal force! Recently I discovered [Why's Poignant Guide to Ruby](http://poignant.guide/), and my oh my, it is a pure *gem*. It is so interesting that I've been reading it for fun, and voila, I'm learning Ruby!

Learning how to program is no different from learning anything else. Maybe I'll get excited about having a goal, which is when I go "What? You can do that?", or I just want to have fun. This helps me stay motiaved during the first phase of learning. If I have a teacher who helps me to keep that process digestable, it's perfect. Once I get to a not-so-beginner level, it becomes easier to repeat this cycle on my own.

## <a href="#how_to_teach_rust" id="how_to_teach_rust">How do we teach Rust to beginners?</a>

This brings us to where my train of thoughts disembarked.

I learned about the existence of the command line about a year ago, and Rust was my first compiled language. I had never learned C or C++ before and had only used Python here and there. I had asked my partner why I should use many functions instead of having just one big mega function last year. So, while I wasn't a complete beginner, I was a semi beginner when I learned Rust three months ago.

Many people like Rust because of its advantages over C and C++, but that motivation didn't apply to me, because I don't know these languages. I had never done systems level programming or known of the computer architecture, so the word memory safety was foreign to me. Similarly, the concept of zero-cost abstractions didn't make sense to me until much more recently.

What helped me get through the initial hump of learning? First of all, I had a very clear goal (make fetch happen in Servo!). Second, once my awesome mentors helped me understand the basics of computers, more and more things started making sense. After that, it was easier to just have fun learning Rust. It was as if I got a giant bag of keys (called Rust) into rooms of knowledge ("memory safety", "parallel computing", "type system", etc.), and I've been having a blast opening random doors and finding treasure to collect. And the more I learn Rust, the more keys I get in my bag. Rust helped me understand many new concepts, and I grew as a programmer a lot over the past three months. I think it's a great gateway language to understanding computer science concepts. I'm proud of how much I learned.

But how would I teach Rust to me 3 years ago?

This is a really difficult question. Teaching Rust to people who've never heard of the command line will be definitely different from the way we teach Rust to people who are experienced programmers. It might come with the cost of not showing all the cool things about Rust in the beginning, because if we do, it will confuse beginners with too many details. I don't think it makes sense to spill jargon (garbage collection, static types, and so on) to complete beginners. Maybe this means we'll have to come up with a way to introduce Rust's core concepts like ownership and borrowing without using jargon (such as "memory"). Maybe we can introduce those concepts through talking about [sharing coloring books with friends](http://jeenalee.com/2016/08/15/sharing-coloring-books-in-rust.html)!

Going a step backward, how can we inspire people who've never learned programming to get excited about it? We should think about what makes people go "Cool! You can do that with programming?" and/or make the process so fun that it's irresistable. Then we teach how to program with Rust. While we learn how to program, magically, we would have learned Rust too. The roles of teachers would be getting learners excited, and keeping things manageably challenging during that first hump of learning.

I don't have concrete ideas on how to do that. But maybe all it takes is some empathy. It's really easy to forget what it was like to learn something new from ground zero. And I think great teachers try hard to remember what it was like. What got you excited about programming that first time? What made you realize programming is useful? What was the most challenging part? What was special about your favorite teacher?

<br>

<p id="ref_0"><small><sup>0</sup> Bread has a lot of starch, which is a chain of sugar molecules. In your spit, you have <a href="https://en.wikipedia.org/wiki/Amylase">amylase</a>, which is an enzyme that helps breaking down starch into sugar. <sub><a href="#0">return</a></sub>
</small></p>

<p id="ref_1"><small><sup>1</sup> When you don't finish the antibiotic prescription, there's a high chance that you helped <a href="https://en.wikipedia.org/wiki/Antibiotics#Resistance">bacteria to evolve to tolerate antibiotics</a>. This creates a class of bacteria that won't be easily killed, which is becoming a serious threat to the public health. <sub><a href="#1">return</a></sub>
</small></p>

<p id="ref_2"><small><sup>2</sup> ...and I got so far. In the end, it definitely mattered. <sub><a href="#2">return</a></sub>
</small></p>

<p id="ref_3"><small><sup>3</sup> They all are a result of <a href="https://en.wikipedia.org/wiki/Lactic_acid_fermentation">lactic acid fermentation</a>! With oxygen, muscle cells can make more energy from carbohydrates. But when not enough oxygen is available (maybe you started sprinting), the cells start making less energy without oxygen, which creates a lactic acid byproduct that causes the soreness in your muscle. The bacteria used in fermented food products are often not capable of using oxygen, so they use no oxygen when making energy (which is fermentation in biology jargon)! <sub><a href="#3">return</a></sub>
</small></p>
