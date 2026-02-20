


# Challenge Description

Flags are usually strings in files, matching the pattern "EPT{...}" where "..." is some text.
Here's an executable that prints the flag for you. Can you find the flag?
(For security reasons, we encrypt the flag before printing)

We're given a binary called flagprinter and told it "encrypts" the flag before printing it — implying we shouldn't just run it and expect a clean answer.

Solution
The hint is right in the description: strings. The strings utility extracts human-readable text from binary files, and since flags follow the pattern EPT{...}, it's worth checking whether the flag is embedded in plaintext despite the "encryption" claim.
Running the command:
bashstrings flagprinter


This dumps all printable character sequences from the binary. Among the standard library references (`libc`, `printf`, `scanf`, etc.), one line stands out immediately at the bottom of the output:


# <img width="576" height="344" alt="image" src="https://github.com/user-attachments/assets/a740f5fd-5bb7-456b-903b-007b78542126" />


## Flag
`EPT{FOOLEDYOUONCELOL}`


The so-called "encryption" was a red herring — the flag was stored as a plaintext string inside the binary the whole time.






