## Setup problems

**1. Node wasn't installed**  
`node -v` returned `command not found` I have anaconda installed, so at first I assumed that the dev environment was there. Installed Node from nodejs.org.

**2. npm permission error (EACCES)**  
The global install failed writing to `/usr/local/lib/node_modules`. Node had installed into a root-owned system folder. I fixed it by setting an npm prefix in my home directory and adding it to PATH, not with `sudo`, because sudo would run the package install scripts as root.

**3. Accidentally using the wrong model**  
At first I used the Opus model with high reasoning effort, this uses lots of my credit from only a couple of test prompts. I then proceed to switch to sonnet model with thinking off.

**4. Skill wouldn't load**  
`openclaw skills list` didn't show it. The file was in the right folder with the right name but `head -5` revealed the YAML frontmatter is missing. OpenClaw identifies a skill by the `name` field, so without it I can't see the file.

## Bugs found during testing

**5. Wrong path and unfinished fix**  
The skill referenced `data/pricing.csv`, which is different where I stored it. I didn`t check thoroughly when changing the path and ended up missing some instances. 

**6. Prompt not showing up on the terminal**  
My first run didn`t show anything, which I thought at first was broken. Turns out it was just not rendering. In the browser dashboard it reached it approval prompt and was waiting for my input. 

**7. macOS permissions**  
Node requested access to iCloud Drive, Apple Music and Photos. The agent does not need any of that. I grant the documents only. 

**8. Approval prompts expire**  
A prompt which ask for an action timed out after 15 minutes, ending the run without approval or logging. For this case where an owner might not check for a couple of hours, this is a design flaw. 

**9. Role Confusion**
The agent got confused and didn`t know if I were the customer or the owner. It even offered to search for a plumber instead of drafting a quote. I added a line that say pasted enquiries are forwarded customer messages. But on enquiry 4, the agent thinks that I am the one asking about my own bathroom. This means there are instructions that contradict each other.

**10. Emergencies skipped flow**   
The agent matched PIPE01 ($280–600) correctly, then deliberately withheld the quote, it decided that active flooding needed urgent fix, not price quotes. My skill listed burst pipe as an emergency urgency and separately kept a list of escalate without quoting cases that didn't include it. The agent merged both concepts. Technically it is not wrong, my spec left the emergency case undefined. That is why the model filled the gap with its own judgment. The same gap meant the enquiry never got logged, since logging is the last step of a flow it had already stopped/exited.

**11. Only 2 of 5 enquiries logged**  
Both logged rows are correct, it even includes a partial row with `unmatched` and empty prices rather than hallucinates the prices. But the enquiry for burst pipe didn`t get logged. Logging should be unconditional and not the last step.

## What worked

**12. Agent did not hallucinate prices, worked twice**  
On purpose: during this enquiry > “do u do gas hot water? need a quote”. There are no suburb and didn`t tell if the customer want to replace or repair. The system correctly identifies this and asked further questions instead of creating a quote. 

By accident: when the file path is wrong, the agent tries to check the price and could not find it. Instead of hallucinate and create a number, it escalated to me.

**13. Ambiguity detection**  
For this enquiry > “Hey mate hot water system thingy cracked, I have no hot water at all. Im at west end, anyone could come today? The things quite old, 10 years old probably”. On the price.csv there are three hot water rows which are repair, replacement, and gas replacement

The agent thinks about what to choose from the three options and decide that it depends on the unit (gas or electric). That is why the agent asked back to make sure.

**14. Partial data saved without guessing**  
On the same enquiry as above, the log shows `unmatched` with blank price fields rather than filling it with random values.

---

