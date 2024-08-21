# 1. Introduction

- **Context**: Many online games incorporate an _**idle check function**_ to monitor player activity. This function is designed to detect when a player has been inactive for a certain period, typically by checking if there has been any input from the player, such as mouse movements or keyboard presses. If the player remains inactive for too long, the game may automatically log them out.

In Starbreak, the _**idle check function**_ will disconnect you from the game if you're inactive for over 20 minutes.

![image](https://github.com/user-attachments/assets/3e317787-0378-4c5e-87a2-b1e51d81aa19)


- **Objective**: This tutorial aims to locate and prevent the _**idle check function**_ from being called to avoid Starbreak from logging you out due to inactivity.

# 2. Static Analysis.
Let's examine the binary file to identify and locate the _**idle check function**_.

## 1. Finding the Idle Check Function

A straightforward way to find this function is by searching for string references in the binary. To do this, try searching for the string displayed in the idle disconnect popup and see if it is referenced in our target function.

![image](https://github.com/user-attachments/assets/a8b89e2e-fbea-41fb-8f81-8a6f9491e407)
We get the result of a constant string located at that specific address.

Double-click the string at the specified address to view it in IDA View. Press ``Ctrl + X`` to list all cross-references.

![7GC5UZ](https://github.com/user-attachments/assets/38800ec9-5cf5-4d8c-81b2-8808129b306c)
Among the cross-references, there is only one function that references the string. In this case,  **``GameClient::updateIdleKick``** is the function we are looking for. Click on it to navigate to the function's assembly code. 

The static function address is located at ``0046A590``. Note this address later, as we will use it in the **Dynamic Analysis** section.

![image](https://github.com/user-attachments/assets/af5e5f7f-20c9-40a7-93ef-6cce80168870)


## 2. Analyzing the Idle Check Function

To make this function easier to understand without examining the assembly code directly, you can use the Hey Rays Decompiler to convert the assembly code into a pseudo-code. Press ``Alt + F5`` to perform the decompilation.

```c
void __thiscall GameClient::updateIdleKick(GameClient *const this, int dt)
{
  int sinceKeypress; // eax
  int v4; // esi

  sinceKeypress = this->sinceKeypress_;
  v4 = sinceKeypress + dt;
  if ( sinceKeypress > 599999 )
  {
    if ( sinceKeypress > 899999 )
    {
      if ( sinceKeypress > 1139999 )
      {
        if ( v4 > 1200000 )
        {
          ChatLog::addMessage(this->chatLog_, "YOU ARE BEING DISCONNECTED FOR BEING IDLE FOR MORE THAN 20 MINUTES.", 1);
          ServerConnection::closeConnection(this->serverConnection_);
          GameClient::doError(
            this,
            "IDLE DISCONNECT",
            "You were disconnected for being idle for more than 20 minutes.",
            1);
        }
      }
      else if ( v4 > 1139999 )
      {
        ChatLog::addMessage(
          this->chatLog_,
          "YOU HAVE BEEN IDLE FOR 19 MINUTES, YOU WILL BE DISCONNECTED IF YOU ARE IDLE FOR 1 MINUTE MORE",
          1);
      }
    }
    else if ( v4 > 899999 )
    {
      ChatLog::addMessage(
        this->chatLog_,
        "YOU HAVE BEEN IDLE FOR 15 MINUTES, YOU WILL BE DISCONNECTED IF YOU ARE IDLE FOR 5 MINUTES MORE",
        1);
    }
    goto LABEL_3;
  }
  if ( v4 <= 599999 )
  {
LABEL_3:
    this->sinceKeypress_ = v4;
    return;
  }
  ChatLog::addMessage(
    this->chatLog_,
    "YOU HAVE BEEN IDLE FOR 10 MINUTES, YOU WILL BE DISCONNECTED IF YOU ARE IDLE FOR 10 MINUTES MORE",
    1);
  this->sinceKeypress_ = v4;
}
```
Let's examine the function. 

First, let's examine the function signature.

The first parameter, ``this``, is a pointer to the current instance of the **``GameClient``** class. The second parameter ``dt`` is likely the delta time, representing the time difference since the last update. This suggests that the function is probably called during every game loop iteration.

Next, let’s examine the function logic. 

There are two local variables: ``sinceKeyPress``, initialized with the value of ``sinceKeyPress_`` from the **``GameClient``** class, and ``v4``, initialized by adding ``sinceKeyPress`` and ``dt`` together. The value of ``v4`` is assigned to ``esi``. And the value ``sinceKeyPress`` is assigned to ``eax`` in the function scope. 

Based on the logic, if the ``v4`` variable exceeds ``1200000``, the client will close its connection to the server and display the idle disconnection popup. 

# 3. Dynamic Analysis

Now that we have located the _**idle check function**_ ,let's modify the program to prevent it from being called.
Let's run the application with **``Cheat Engine``** attached to the process. 

## 1. Finding the Idle Function in Game Memory

Go into ``Memory View``, press  ``CTRL + G`` and input the address ``0046A590`` for **``GameClient::updateIdleKick``** address ``0046A590`` to navigate to the start of the function.

![P1kTN3](https://github.com/user-attachments/assets/1ba0c4ae-1af6-4f67-9147-94eb6ccb8ef7)

## 2. Finding the Local Variables.

![4gv3yl](https://github.com/user-attachments/assets/019bb420-c943-40b0-9df8-66a5b77dd996)

Here is the assembly code that initializes the local variables. The previous section shows that the value of ``v4`` is assigned to ``esi``, and ``sinceKeyPress`` is assigned to ``eax`` within the function scope.


``ecx`` likely points to the **``GameClient``** class, with ``6C`` offsetting the ``sinceKeyPress_`` property.

![image](https://github.com/user-attachments/assets/778c3332-aabf-435c-999d-c2c742889ab1)

To verify this, we can examine the structure of the **``GameClient``** class in IDA Pro and confirm that ``6C`` offsets the ``sinceKeyPress_`` property.

## 3. Debugging

Here’s a revised version of the sentence:

Let’s debug the function to find the pointer address to the **``GameClient``** class, allowing us to access the ``sinceKeyPress_`` property. Remember that since the class is dynamically allocated, the pointer address will change each time you open the game.

![image](https://github.com/user-attachments/assets/1efc25de-96b7-4995-9f7c-126958e3ad0c)

Insert your breakpoint, and the program will stop at that breakpoint. We can see the registers when the breakpoint is hit. 

The **``GameClient``** class pointer address in ``ecx`` is ``1AB05B50``, and if we add the offset ``6C``, we should be able to access the ``sinceKeyPress_`` property ``1AB05BBC``.

![QDu3hH](https://github.com/user-attachments/assets/0c5e3687-d9f1-442f-9afd-19facbd84105)

Let’s monitor the value as the game runs to verify that we have the correct address. Don’t forget to toggle your breakpoint off!

## 4 Testing
If we unfocus the game, we can see that the value is increasing, and if we refocus and add keyboard input, the value is reset near 0. 

Let's do another test and set the value to ``899999``. Now, a warning text appears in the chatbox.

![image](https://github.com/user-attachments/assets/a039394a-cd25-4434-913f-32ff395024b6)

Awesome! Now we have the memory address where the value is used to check if you have been idling too long. 



