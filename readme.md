# Tutorial
In this series of tutorials, I'll show you how to _**reverse engineer**_ the game **Starbreak**. 

# What is Starbreak?
StarBreak is a unique skill-based action platformer MMO where you explore strange sci-fi worlds alongside dozens of other players, kill legions of dangerous aliens, and fight epic boss battles.

# Why?
This game includes many features, including monster drops, loot, teleportation, and more. What if we could develop a script to automate some of these actions and make our gameplay experience smoother? Not only would it simplify gameplay, but it would also provide a valuable learning experience in reverse-engineering games. 

# Prerequisites

To follow along with this tutorial, you should have:

- A basic understanding of x86 Assembly
- Experience using a disassembler (e.g., Ghidra, IDA Pro)
- Knowledge of C programming
- An understanding of low-level programming concepts
- Familiarity with Cheat Engine and its basic functionalities

## 1. Game and Disassembler Installation
Before you start this tutorial, you will need to install the game and a disassembler.

I'll be using [IDA Pro](https://hex-rays.com/ida-pro/) (a paid program) for the rest of the tutorial, but free alternatives like [Ghidra](https://ghidra-sre.org/) are also available.

The game is available on Steam. You must have a Steam account to install the game.

## 2. Disassemble the Game Executable

1. Open **IDA Pro** and ensure you use the x86 version; otherwise, you'll receive a warning. Then, click **"New"** to disassemble a new file.

![image](https://github.com/user-attachments/assets/ead38147-754b-45b5-93f1-5dac5c2e3b66)

2. Next, locate the game's executable file. Using Steam, you can easily navigate to the folder where the game is installed.

![image](https://github.com/user-attachments/assets/1468aa33-ac18-4c6a-a406-ac9aea7e7b58)

Go to your game library, search for Starbreak, and right-click on it. In the dropdown menu, select "Properties" to open the game's properties window.

![image](https://github.com/user-attachments/assets/e4a07cb2-2978-4656-9a83-37b903ffd84a)

Go to the **Installed Files** tab and click **"Browse**" to open the installation folder. 

![image](https://github.com/user-attachments/assets/f068b579-b12a-4d69-adab-5959b962db0c)

The game's executable file is named ``mvmmoclient.exe``. 

4. Drag the executable file into IDA Pro, which will begin disassembling and processing.

![image](https://github.com/user-attachments/assets/b89f7ee0-925a-4d46-960e-8037b169a394)
Your **IDA Pro** should look similar to the screenshot above. 

## 3. Information Gathering

Before we start _**reverse engineering**_ this game, let's first gather publicly available information about it, such as the programming languages it uses and any libraries it uses.

Search for ``“What programming language is Starbreak based on?”`` on Google, you’ll find an interesting blog post by the creator of Starbreak at the top result.

![image](https://github.com/user-attachments/assets/408eb7d9-da2f-46c5-b94a-48c3780af704)
[Read the Blog Post](https://www.crunchy.com/emscripten-a-perfectly-cromulent-compiler.html)

TL;DR: The server and client are written in C++.

## Finished
Now that we have set up our environment, we can begin dissecting the game and exploring what we can incorporate into our scripts.
