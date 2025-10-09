A walkthrough of recording actionable time metrics for an incident response team.

<!--more-->

## A bit about me and what made me write this

I’ve been working for one of the major US airlines for a few years now and I've had the amazing experience of seeing much of what is under the blue team net. From detection engineering to policy writing I've gotten to dip my toes in many of the cyber subcategories. 

About two years ago now we decided to in-house our IR program and I led the proccess of building and defining the whole responding-to-incidents thing. All the while I worked as a responder feeling the impact of what I was building.  After months of writing and writing until we were so sick of writing that we wrote for another month, my head turned to metrics. I blame my boss for (thankfully) drilling the saying “you can’t manage what you can’t measure” into me.  

Today, we’ve gotten these metrics to the point where all stresses and performances of the IR team reflect themselves automatically. I can’t stress enough how important this is for coordination with leadership. Saying “please help and get me more resources” will always be met with a “why”. We want to change that to “Here’s a big spikey part of a graph that shows how strained my team is, either fix it or we can’t respond well.” (No, that is not how I talk to my boss). We want actionable data that give business leaders a choice based on risk and resources.

## So How’d I Start?

The same place everyone does, looking it up. After a bit of digging I was disappointed in the industry wide definition of IR metrics. Fun experiment, google “time to detect incident response”. Here’s the first 3 definitions I got from the search engine gods. 

1. MTTD is defined as the average amount of time your team needs to detect a security incident.
2. The mean amount of time it takes for the organization to discover—or detect—an incident.
3. MTTD (Mean Time to Detect) measures the average amount of time it takes to identify an issue.

So the time to detect is the time it takes for us to detect an incident. I truly did not see this coming...
<img title="nooo" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/nooo.jpeg?raw=true" alt="" align="left" width="863" height="90" data-align="right">

While it's not wrong it's not specific enough. Many of the articles and definitions all give the vibe of “mhmm yes you want to detect better and respond quicker”. Very few actually define where the goalposts are for the start and the end.

To build my actionable metrics I decided to start from scratch and draw our own lines in the sand. I’ll share the specific metrics which have helped me monitor the pulse of how my team is performing. 

This by no means is a perfect model but perfect is the habitual enemy of the good. Get some numbers down and start measuring. The story will start to speak for itself before you know it!

## Cyber Detect and Response Chain

First we need to understand what we are trying to measure. Assume that all incidents will follow a chain like the one below. Yes I know they all don’t but if we try and hit all the edge cases we’ll never get started. This chain represents a case where there is some dwell time and automations aren’t doing their thing. Take the following example:

“A phishing attack that leads to an account being compromised. Two weeks later the actor takes action that creates a SIEM detection. This detection is then forwarded to the SOC which will process the alert and a user will manually quarantine and remediate the account.”

<img title="Blank_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/Blank_Chain.png?raw=true" alt="" align="left" width="863" height="90" data-align="right">

In order to get a consistent start and end we need explicit definitions for A-H above. 

###### **A. Initial Compromise:** The first step in the killchain.

- Ex: Malware Downloaded to a System, Phishing email coming in, USB drive plugged in
- Note that you may not know what this time is until a case is fully closed. Actors can lay dormant for months before making a move. 

###### **B. Unseen Activity:** All activity from the threat actor that is not responded to by the security team.

- This includes logs and alerts which are noticed by a tool but never picked up by the team. 

###### **C. Reaction Event:**  Initial log or alert that will trigger a response process

- Ex: Network log that goes to a SIEM and results in a detection

- Ex: The timestamp an EDR blocked a piece of malware which makes a case that gets reviewed by an analyst

- **If there is no response to a detection, alert, log, or other means the Reaction Event has not occurred.**

- Logs which are discovered during an investigation which pre-date the Reaction Event do not count. Those should fall in Unseen Activity.

###### **D. Alert Hits Queue:** The time the alert/detection/etc arrives on the platform a human will respond to.

- A tool like a NTA or ASM is configured to send alerts via Email to the IR team. Timestamp D is the time when the team receives the email.
- SIEM fires a detection that gets sent to a SOAR which then creates an alert that an analyst analyzes.  Timestamp D is the creation time of the alert in the SOAR.

###### **E. Human Acknowledgment of Alert:** The time an analyst first picks up the alert/detection they will respond to

- Button for acknowledgement in a case management platform

###### **F. Investigation:** Time spent triaging, information gathering, etc

###### **G. Mitigation:** Time when the threat is fully mitigated by manual response actions.

- Team members quarantine an account.
- Team members manually isolate a host.

###### **H. Closure:** Time which an incident is marked as closed and tied up all nice with a big red bow on it.

Is this a gross generalization? Yes. If you’ve worked more than 5 incidents in your life you know that sometimes points overlap. For example, end users downloading malware (Which remains one of America’s favorite past times). A half decent EDR tool should pick up on recent strains and block it the second it hits the system. In this case A and C are the same and B is 0. 

Soooo please bear with me and ignore the cases which don’t fit into the bucket for now.

## What Are We Trying to Measure?

Before we jump right in and start throwing acronyms around we need to ask what we’re actually trying to measure. As an IR lead or IR manager the following questions jump to mind around speed:

1. Are we hitting SLAs?

2. Are there any slowdowns in the data/alert transit?

3. How efficient is my team?

4. How quickly can we contain threats?

5. Are we detecting threats in our environment quicker?

Leadership on the other hand will likely have a more general view. Is my IR team getting quicker at detecting, faster at mitigating, and more efficient with each case?

## Drawing Deltas

Cool, problem defined. Lets run through those questions to show how the detect and respond chain can get us some answers.

### Are we hitting our SLAs?

My goal here is to strictly measure human performance. From the time that an analyst could acknowledge an alert to the time an analyst did acknowledge an alert. 

This can be seen in the delta between “Alert Hits Queue” and “Human Ack of Alert”. This gives us our first TT metric of the day! 

**Time to Acknowledge (TTA)** - The time between a notification on the platform the team responds on and the analyst acknowledgement of the alert

<img title="TTA_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/TTA_Chain.png?raw=true" alt="" align="left" width="863" height="130" data-align="right">

This should not include any time that isn’t measuring strictly human performance. This gives us a clean number we can monitor to give confidence in our ability to process all the alerts. Seeing it rise can be an early indicator the team is experiencing alert fatigue (ask me how I know). It also lets us track the SLA around how quick the team gets to alerts. 

### Are there any slowdowns in the data/alert transit?

In many cases, the initial detection that fires isn’t on the platform where analysts are reviewing them. Whether SIEM/SOAR or other case management system, there is some transit time for the alert. 

Having a pulse on this for your different alert sources is crucial for the IR team. You’re not watching security cameras or anything else with real time views. There will be some form of delay, either seconds or minutes and hopefully not hours. An analyst picking up the alert needs an expectation for how far back the behavior is.

**Processing Time (PT, or TTP if you really need another TT number)** - The time between the initial alert which will trigger a response process and the alert hitting the system the team will see it on.
<img title="PT_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/PT_Chain.png?raw=true" alt="" align="left" width="863" height="130" data-align="right">

Processing time’s end also marks the transition from automated process to analyst response.

### How Efficient is My Team?

Again this should be a strictly human process. We are measuring our efficiency in triaging and coming to conclusions.

**Time To Close (TTC)** - The time from when an analyst picks up an alert to the time when an analyst closes the alert.
<img title="TTC_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/TTC_Chain.png?raw=true" alt="" align="left" width="863" height="150" data-align="right">

I want to add the caveat that I'm torn on driving this metric down. We want to speed up processes where we can but I don’t want to rush the team or an investigation just for a good looking chart. I’d recommend stressing to leadership that the number is expected to maintain, not decrease. 

### How Quickly Can We Contain Threats?

In a perfect world I'd like this number to be 0. Realistically most orgs will have a tolerance for what they can block automatically and what they need authorization for. No one is going to allow you to turn the server off that keeps the website up without human supervision.

So, let's cut the metric in half. For all automatic mitigations (think EDR, Zero Trust solutions, FW blocks, etc) where the malicious behavior is stopped during initial access, the value is 0. Looking at our fancy chart this means (A) and (G) or (C) and (G) are the same. These cases are best recorded as an overall trend number.

The rest of the cases should have no auto-mitigation, which means the IR team will be doing a bit of clean up. 

**Time To Mitigate (TTM)** - The time from the IR team picking up an alert to the time the case is fully mitigated.

<img title="TTM_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/TTM_Chain.png?raw=true" alt="" align="left" width="863" height="130" data-align="right">

Like TTA this should be a strictly human measurement. We want a gauge on how enabled our team is to mitigate threats when they’re pressing the button. This should go down over time as the team sures up their procedures and are empowered to take more decisive actions.

### Are we detecting threats in our environment quicker?

Saving the best, and trickiest, for last. Spoiling it now, it's Time to Detect. The metric that everyone throws around as the goal post to lower but no one seems to have a clear idea on measuring. This could be an article all in its own if you include hunting and other disciplines but i’ll try and keep it simple here.

**Time to Detect (TTD)** - The time between initial access to the initial alert which will trigger a response process.
<img title="TTD_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/TTD_Chain.png?raw=true" alt="" align="left" width="863" height="130" data-align="right">

Now let me explain why this is a terrible metric.

##### TTD Case 1:

Someone downloads a known malicious file like a RAT which immediately gets blocked by a EPP. This block then gets reported to an analyst. *This encompasses all security incidents which get reported during the initial access technique/start of the attack chain.*

In these types of cases the TTD is always 0. There is no downtime between the start of the incident and the time when a security tool reports the behavior to an analyst. 

##### TTD Case 2

An account is taken over on January 3rd. The actor lays dormant for two weeks and then downloads a piece of malware which gets spotted by the endpoint agent. The agent then creates and sends a case to an analyst. *This encompasses all security incidents which are detected by a tool later in their attack chain post initial compromise. This implies other activity that could have been detected but was not.*

Here the time to detect relies on the actor performing an activity that is bad enough for a security tool to detect it. In the example the time to detect is two weeks. These cases will contain a lot of variation and can range anywhere from minutes to months.

##### TTD Case 3

A machine is infected with a piece of malware that the EPP does not find. In a threat hunt performed by the security team, logs in the SIEM are found that could indicate a compromise. These logs are then sent to a CIRT team who declares an incident. *All security incidents where a person is the reason the investigation starts.*

Here the time to detect relies on humans coming across the activity. This time will have the most variation and like example 2 can range from minutes to months.

#### The TTD Dilemma

It would seem TTD is either 0 or a widely varying number. 0s representing proper security controls and the other numbers representing a detection gap. Aka we’ve got two distinct items to measure but we’re using one name. 

If we find detection gaps in the environment the hope is we can add new alerting to help close it. Take a new initial access technique like those one note files that got popular a few years back. Someone falls for it and two weeks later a malware alert pops up. As an after action we add a detection to catch those one note files moving forward. 

If it took us two weeks to notice the first time, the second will be instant. Averaging these two TTD to one week isn’t the correct story. We’ve improved our posture and gotten better!

Because of this, metrics like number of new detections, ATT&CK coverage, and number of blocked cases are a much better indicator of how we’re doing at detecting threats.

Moral of the story? Yes TTD exists but it's a silly metric measuring way too many things. We’re better off leaning on metrics that aren’t time based to actually answer the question, “are we getting better at detecting?”.

## Putting It All Together

<img title="Final_Chain" src="https://github.com/amalone341/amalone341.github.io/blob/main/assets/image/Final_chain.png?raw=true" alt="" align="left" width="863" height="210" data-align="right">

These deltas are the specific gaps which have given me actionable and consistent metrics for my team once implemented. I’d be surprised if there is full agreement on where I drew these lines or whether or not TTC should be TTR or TTD or any other fun acronym.

The key is to be consistent for your own team! Spend the hours getting waaay into the weeds about definitions and metrics. Arguing about where those lines should be and what to record will force clarity in the definitions and result in a framework that works for you! If you do decide to use my chain and lines please make sure you understand where the timestamps are for each one of the steps.

If you liked this blog and want more in-depth opinions on IR metrics please let me know! I’ve got a whole blog bouncing in my head on just TTD alone and we didn’t touch on non-time metrics here. Thanks for hanging with me and I hope this helps a bit for all my blue hats out there!

