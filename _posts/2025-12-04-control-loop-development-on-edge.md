---
title: "Control Loop Development on the Edge"
date: 2025-12-04
categories:
  - Technical
  - Learning
tags:
  - Edge Computing
  - PID Controller
  - Game Development
  - Research Methods
excerpt: "Exploring PID controllers and edge computing through the lens of game development - learning control theory by building an interactive game on the Play Date console."
header:
  image: /assets/images/2025-12-04-pid-controller/playdate-console.png
  teaser: /assets/images/2025-12-04-pid-controller/playdate-console.png
---

# Control loop development on the edge

I've recently been ramping up my knowledge on deploying power control algorithms to edge device platforms. These algorithms must monitor and maintain the power consumption envelope of a system at millisecond latency with high accuracy, and they must run on a device embedded in the power system itself.

This represents a significant shift for me, in both domain knowledge (control loops) and system architecture (edge computing).  In my past roles I've worked on operational platforms for Distributed Energy Resource (DER) asset monitoring and diagnostics. In that context, the statistical models and software I worked on ran in a centralized cloud hosted environments. The data they consumed were sourced from pipelines calling distributed assets reporting at a minute interval granularity.  Their objective was to issue analysis and diagnostics to an operations platform.

Monitoring a power system's state and issuing commands on the sub-second scale, hosted on an edge device, is a brand new space to me.

# Analogous research

There is a method in software and industrial design called analogous research that argues there is often value in looking outside of your industry to find analogues to the problem you are trying to solve.  They may use very different terminology and arise in very different contexts, but they share some fundamental similarities that make solutions and learnings transferable.

The oft cited example is the case of F1 pitstop crews inspiring the redesign of operating theaters. The paper can be found [here](https://onlinelibrary.wiley.com/doi/10.1111/j.1460-9592.2006.02239.x) . IDEO has developed this concept into a full blown methodology, which you can read more on [here](https://www.ideou.com/blogs/inspiration/activity-explore-analogous-inspiration)

![IDEO analogue inspiration methodology diagram showing how to explore similar problems in different domains](/assets/images/2025-12-04-pid-controller/ideo-analogue-methodology.png)

## Motivations for gaming console software as an analogue
I'm not implementing it in full here, but as I started looking at the problem of control loops on the edge, I was struck by an analogy that could be really handy: building games for console devices. Here's my thinking:

### Motivation 1: Power control loops look a lot like the Game Loop
If you building a power management solution, you will need a process that continuously monitors the state of a system, reacts in real time to any changes in that state according to some cost function, and updates the state of the system using kind of control function.  These processes are called control loops. They tend to operate at known frequencies (measured in Hz) which determine, in part, the speed at which the system can react to change and adjust the systems state.

![Generic control loop diagram showing the feedback cycle of monitoring state, computing control signals, and updating system state](/assets/images/2025-12-04-pid-controller/control-loop-diagram.png)

In the case of a game, a game's engine is basically a big old loop that is continuously updating the game's state while monitoring for user inputs.  Imagine Pac Man updating the position of Mr Pac Man , all the ghosts etc, while also checking to see if you have moved the joystick. This also tends to run at a target frequency which the user often experiences in terms of frames per second (fps) . A fun history here, in the old days, game loops simply ran as fast as the hardware would let them , meaning old games ran too fast on new hardware. These days, game loops also own normalizing the passage of real time in a [game](https://gameprogrammingpatterns.com/game-loop.html). For more on game loop architecture, see [this OpenGL reference](https://www.oreilly.com/library/view/opengl-game-development/9781783288199/ch01s02.html).

![Game loop diagram illustrating the continuous cycle of processing input, updating game state, and rendering to screen](/assets/images/2025-12-04-pid-controller/game-loop-diagram.png)

But either way, the point is we want a continuously updated state running on a small device that is listening for updates and issuing commands (to either manage power, or update a display, or whatever).

### Motivation 2: Edge DevOps looks a lot like console DevOps
The other thing I've noticed is that there are some very unique and interesting challenges with hosting an efficient development cycle for an edge device that don't really come up in cloud hosted services, but I'd wager definitely do come up in some game development contexts. Specifically;

* **Your production environment is a lot more resource constrained  than (or at least very different from)  your development environment:** This isn't always true for edge or gaming, but I'd say it comes up often in both. While our development environments are built for versatility and performance, edge devices and gaming consoles are built for specialized performance and portability, favoring lower power consumption, better thermal efficiency, and more compact designs. Put in plain terms, stuff that runs on your laptop may not run so well (or at all) on your target device. Which brings up a further parallel;

* **Your production environment isn't in the cloud:** Wanting to test and stage your software in a representative platform isn't a new problem. That is what most CI/CD pipelines are built for. However, most  CI/CD frameworks assume  deployment via cloud platforms like AWS or Azure, which makes provisioning and deploying to resources much easier. But what if your platform is a specific device, a 'thing', that may or may not be connected to the internet?  What does the dev cycle look like then?

### Motivation 3: Curiosity is a lot more fun than anxiety
Finally, I think its also worth reflecting on what games can teach us about coping with ambiguity and solving complex problems.  As anyone who has worked in startups can attest to, building a 0 to 1 product is intellectually and emotionally demanding. Having spent a while in this space, I've found one of the most critical areas for my professional growth is responding to that ambiguity with curiosity and optimism, rather than anxiety and dread.

Easier said than done I know. I'm not the first to notice this. There's compelling evidence that curiosity and anxiety are just bifurcations of the same pathways in the brain's response to novel situations. You can read a meta analysis of research in this domain [here](https://pmc.ncbi.nlm.nih.gov/articles/PMC11220156/), I've provided the author's high level interpretation of what happens to a brain thats presented with a novel problem or situation. The bottom line is that if the amygdala is allowed to have a say in the response, you will usually end up in anxiety mode. This checks out with most of the other works I've read on anxiety as a condition and how to manage it ([e.g.](https://marthabeck.com/beyond-anxiety/))

![Neuroscience diagram showing the brain's response pathways to novel situations, illustrating how curiosity and anxiety are related but divergent responses](/assets/images/2025-12-04-pid-controller/brain-response-curiosity-anxiety.png)

What's striking about games is that many of them *intentionally simulate ambiguity and problem solving*. Yet with games, we deliberately seek them out, and we enjoy the ambiguity. Go figure. What this suggests to me is that games can be a useful way of re-packaging novelty and uncertainty in a way that makes the brain less likely to freak out and switch into anxiety mode.

# Gamifying a control loop on a Play Date console

## The Play Date platform

I've been a fan of [Panic](https://panic.com/) ever since they release [Untitled Goose Game.](https://goose.game/) I love their style and esthetic, and I think they produce solid products and code.  Recently they have released their own console , the [Play Date](https://play.date/), along with an [SDK](https://play.date/dev/) that allows you to jump right into game development, primarily using the Lua coding language. It ticks all the right boxes for an analogue to my domain: it has limited resources compared to my dev environment, it operates primarily offline with some limited Over The Air (OTA) and USB enabled deployment patterns. It also has a quirky 'crank' control interface which I think might be a handy way to simulate inputs to a control loop.

....These are all the things I told my partner when justifying why a fully grown man should spend 200 bucks on a retro gaming device...

![Play Date handheld gaming console with distinctive yellow crank controller](/assets/images/2025-12-04-pid-controller/playdate-console.png)

## A basic control loop

![Basic PID controller block diagram showing the three components: Proportional, Integral, and Derivative terms](/assets/images/2025-12-04-pid-controller/basic-pid-controller.png)

Given I am a rookie in control loops I am going to use/learn about the vanilla latte of control loops: The Proportional–integral–derivative controller (PID controller). This model has been around in analogue format since the 1920s. As the name suggests, it's composed of three components:

- **Proportional term**: A coefficient that responds to the current error - how far off you are right now from your target. The larger the error, the stronger the correction. Think of it as pushing harder when you're further from your goal.

- **Integral term**: Accumulates error over time, addressing persistent bias or steady-state error that the proportional term alone can't eliminate. If you're consistently missing your target by a small amount, the integral term builds up and applies increasing correction until that offset disappears.

- **Derivative term**: Responds to the rate of change of the error, essentially predicting future error based on the current trend. This acts as a damping mechanism, reducing overshoot by easing off the correction as you approach your target.

Together, these three terms combine to form a control signal: **u(t) = Kp·e(t) + Ki·∫e(t)dt + Kd·de(t)/dt**, where each component addresses a different aspect of control - present error, past accumulated error, and future projected error.

The art of PID tuning lies in finding the right balance between Kp, Ki, and Kd coefficients for your specific system. I've enjoyed some similar toy versions of the PID controller online that demo this ([e.g.](https://chev.me/pid-demo/)) and wanted to put together something similar on the Play Date.

### The high level game design

I'd like the game to demonstrate how a PID manages a system to a specified target and what the role of the P, I, D parameters are. The user should be able to adjust what the target is that the control loop is 'chasing' in real time (the 'crank' function on the Play Date is perfect for this) and they should be able to adjust the PID parameters to observe changes in performance. I've sketched out a little outline below

![Hand-drawn game design sketch showing the PID controller interface with graphs, parameter controls, and crank input](/assets/images/2025-12-04-pid-controller/pid-game-design-sketch.png)

## The Play Date platform and SDK
Playdate provides an SDK with Lua and C APIs, documentation, and crucially for this project, a **simulator** that allows you to test code on an emulated device.

![Play Date SDK simulator showing the development environment running a game in an emulated device window](/assets/images/2025-12-04-pid-controller/playdate-sdk-simulator.png)

### Faster start up with templates
In addition to the SDK, Play Date's dev website signposts to some handy templates and how-tos to help you get started. These are especially handy if you are new to playdate project structures, build pipelines, the Lua coding language or all of the above. I leaned heavily on SquidGod's template which you can find [here](https://github.com/SquidGodDev/playdate-template) .

![SquidGod's Play Date template repository on GitHub showing project structure and setup instructions](/assets/images/2025-12-04-pid-controller/playdate-template-github.png)

### Continuous deployment with emulators
In addition to being able to jump right into defining the game loop in Lua, the combination of the SDK and some nicely defined templates for VSCODE meant I could compile and deploy the game to a local emulator with a single click. I could then test the game without having to deploy to a real device. This saved a lot of time and shortened iteration cycles.

# Results and conclusions

## AI helps with the basics

Lua is a pretty forgiving language if you are familiar with Python. I worked with Claude to put this program together, though I did find that by the end of the project, I had pretty much rewritten most of the PID controller logic, and much of the plotting logic.  I also iterated through several prompts which I would say were comparable in length to the actual code base.

Its a solid reminder of what AI is (currently) great at, and what it is (currently) useless at:

Do use AI for
* Understanding a new coding syntax quickly
* Boilerplate functions  / well defined modular additions to code

Don't use AI for
* Implementing theoretical models in code
* Front end design / rendering config (it can't really 'picture' what you need and the results are aesthetically awful)

Also, a multi agent setup that involves a code review helped improve quality a lot.

You can check out the resulting code [here](https://github.com/InternetGareth/play_date_controller)

##  Multiple frequencies need to be managed
It was interesting to see how quickly the question of how to maintain a realistic and constant time scale in the game came up.

The SDK is in charge of the frequency of calls to playdate.update() (typically 20 to 50Hz). However, the in-game physics is set to update at time intervals of 0.05 seconds (i.e. 20Hz). Therefore there was code necessary to enforce screen updates to that frequency. This was done by checking that the elapsed time since the last screen update is greater than or equal to UPDATE_INTERVAL (0.05 seconds), essentially passing over any calls to playdate.update() that occurred before that time. I doubt this gets you to a perfect 20Hz, but it's close enough.

## Emulation speeds up the development workflow

As hinted at above, a major feature of the Play Date SDK is the ability to evaluate a build on a local emulator. The ability to quickly trigger a build pipeline that launches the emulator in less than 5 seconds meant that iterations on tricky stuff like front end design, game play, responsiveness etc. became very efficient.

![Development workflow diagram showing the rapid iteration cycle enabled by the Play Date emulator](/assets/images/2025-12-04-pid-controller/development-workflow.png)

## Practical insights on PID calibration
The resulting game allows the user to play with with PID coefficients and observe performance changes. Below are a few examples.

### Ki : Integral co-efficient causes over-shoot
When the Ki term is too high, the control loop overcompensates for past errors by 'over-shooting' corrections. The Mean Absolute Error in this case was at around 109Watts.

<video width="100%" controls autoplay loop muted playsinline>
  <source src="/assets/images/2025-12-04-pid-controller/pid-ki-overshoot-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<p class="video-caption"><em>PID controller with excessive integral gain (Ki) causing overshoot. The system overshoots the target setpoint repeatedly, resulting in a Mean Absolute Error of approximately 109 Watts.</em></p>

## Kd: Derivative coefficient causes oscillation
When the derivative term is set too high, the system over-compensates for the rate of change in error. As a result it will start oscillating, driving very high errors.  The MAE here approached 1000Watts.

<video width="100%" controls autoplay loop muted playsinline>
  <source src="/assets/images/2025-12-04-pid-controller/pid-kd-oscillation-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<p class="video-caption"><em>PID controller with excessive derivative gain (Kd) causing oscillation. The system overcompensates for rate of change, creating unstable oscillations with a Mean Absolute Error approaching 1000 Watts.</em></p>

## Kp: Proportional term and others are tuned for best performance
Finally, here is an example of my best efforts to play around with all three terms to achieve the lowest error possible.  The actual power (the solid line ) is tracking reasonably well against the power set points (the dotted line). The MAE is 28W.

<video width="100%" controls autoplay loop muted playsinline>
  <source src="/assets/images/2025-12-04-pid-controller/pid-optimal-tuning-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
<p class="video-caption"><em>Well-tuned PID controller with balanced Kp, Ki, and Kd coefficients. The actual power (solid line) tracks the setpoint (dotted line) closely, achieving a Mean Absolute Error of only 28 Watts.</em></p>

# Key Takeaways

This project demonstrated the value of analogous research in learning new technical domains. By reframing edge computing and control loops through the lens of game development, I was able to:

1. **Understand control loops intuitively**: The game loop analogy made PID controllers more approachable and easier to reason about in real-time contexts.

2. **Navigate edge DevOps challenges**: Working with the Play Date's emulator and SDK provided hands-on experience with deployment patterns that translate directly to industrial edge devices.

3. **Learn through play**: Gamifying the learning process reduced anxiety around unfamiliar concepts and made iterative experimentation more enjoyable.

4. **Recognize AI's current limitations**: While AI excels at syntax translation and boilerplate code, it still struggles with implementing theoretical models and front-end design - areas requiring human intuition and iteration.

The resulting PID controller game is available on [GitHub](https://github.com/InternetGareth/play_date_controller) for anyone interested in exploring control theory concepts in an interactive format. Sometimes the best way to learn something new is to make it fun.
