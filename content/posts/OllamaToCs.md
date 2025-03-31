+++
date = '2025-03-15T11:26:12-04:00'
draft = false
title = 'Connecting to Ollama running on Raspberry Pi'
tags = ['Raspberry Pi', 'Ollama']
categories = ['programming', 'artificial-intelligence']
+++

## Prerequests
Here are the instructions for connecting to Ollama running on my Raspberry Pi from my C# application. I will provide step-by-step instructions for installing Ollama and connecting it to a C# application. Please ensure that your Raspberry Pi is already running some version of Linux and that you have Visual Studio installed on your computer.


## Installation
Let us start by installing Ollama on your Raspberry Pi (RbP). I've named my RbP rbpi-5-1, and if you see "rbpi-5-1" in the rest of the document, I'm indicating my RbP.

We'll start by visiting [Ollama's website](https://ollama.com/) and copy the curl script provided for Linux installation.

```sh
curl -fsSL https://ollama.com/install.sh | sh
```
Run the above curl command on your RbP, and it takes care of installing Ollama on your RbP. The first step is completed! It is time to check out the installation. We invoke Ollama with the command:
```sh
ollama run {model_name}
```
Several models that are available, each specializing in a different subject. For example, *codellama* focuses more on coding, while *wizard-math* focuses on math and logic problems.

With my vast experience using Ollama, which is 3 hours at the time of this writing, gemma2 is the best for me. It has a wide range of applications across various industries and domains. It is good in 

 - Text Generation
 - Chatbots and conversational AI
 - Text summarization
These are my needs as of now, and hence  ***Heil gemma2!***
### Testing
Having installed Ollama, let us do some basic testing. Invoke Ollama's CLI frontend with the command:
```sh
# Let's use the 2b model - the smallest available!
ollama run gemma2:2b
```
If this is your first time running this command, it is time to make a cup of coffee. It has to download the model, parse it, and eventully you will get the prompt.
```
>>> Send a message (/? for help)
```
Ask your first question: if you go blank like me, here are some examples.
```
Who was the first person to land on the moon?
Why is the blood red?
Why is the sky blue?
```
If your new chatty friend answers correctly, you are good for the next step. *If not abort!*

### Service fix
Ollama is now running as a systemd service. Hence, the environment variables should be using systemctl: we need to allow access to other machines to use your ollama instance, and hence, we need to open the port for external requests. I tried to follow Ollama's FAQ verbatim, and I failed. So I suggest you follow my suggestion.
```sh
sudo systemctl stop ollama.service
sudo systemctl disable ollama.service
cd /etc/systemd/system
```
Now, in this folder, we are going to edit the file  <u>**ollama.service**</u>. In this file, under section Service, add the following line:
```
Environment="OLLAMA_HOST=0.0.0.0"
```
After editing, my file looked as follows: yours will be different, especially in the path, but it gives you an idea of how it should look.
```
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
Environment="OLLAMA_HOST=0.0.0.0"
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/home/sridh/bin:./:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games:/home/sridh/bin:/home/sridh/.dotnet/tools"
[Install]
WantedBy=default.target
```
Restart ollama as follows:
```sh
sudo systemctl enable ollama.service
sudo systemctl restart ollama.service
```
I prefer using the restart command over the start command. Feel free to choose whichever option you prefer. I won't judge you!
The last thing we'll be doing when we are still logged in your RbP will be checking if you can access your RbP from your computer. In my case, in my browser, I navigated to my [rbpi-5](http://rbpi-5-1:11434/) using the following link:
```
http://rbpi-5-1:11434/
```
I got a beautiful welcome message:
```
Ollama is running
```
We can now ignore the terminal window for now and work with Visual Studio.

## Cleint Application
We are going to use a Nuget package, OllamaSharp, to talk to our local LLM server. I wanted to use Microsoft's Semantic Kernel to connect to Ollama, but I need to do more work to make it happen. For now, let us be happy with this package, and when we hit a wall with it, we'll try Semantic Kernel once again.
In Visual Studio, create a console application and add the following packages:
```
Install-Package Lyndychivs.Fibonacci
Install-Package OllamaSharp
```
Fibonacci is more of a gimmick, but why not? It makes the terminal behave a little differently from regular chat applications.
The following code does not require much explanation; once you read it, you'll know exactly what it is doing.
```cs
using Fibonacci;
using OllamaSharp;
using System.Numerics;
using System.Text;

var endpoint = new Uri("http://rbpi-5-1:11434");
var modelId = "gemma2:2b";

var Fibonacci = new FibonacciRecursive();
List<BigInteger> sequence = Fibonacci.GetFibonacciSequence(15);
int sequenceCount = sequence.Count();
Random rand = new Random(DateTime.Now.Second);
var ollama = new OllamaApiClient(endpoint, modelId);
var chat = new Chat(ollama);
Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("AI: Now How can I help?");
Console.ForegroundColor = ConsoleColor.Yellow;
Console.Write("User: ");
var helpTopic = Console.ReadLine();
var outMsg = new StringBuilder();
while (!string.IsNullOrEmpty(helpTopic))
{
    outMsg.Clear();
    Console.ForegroundColor = ConsoleColor.Green;
    Console.Write("AI: ");
    await foreach (var answerToken in chat.Send(helpTopic))
    {
        await Task.Delay(50);
        outMsg.Append(answerToken);
        if (outMsg.Length >= sequence[rand.Next(0, sequenceCount)])
        {
            Console.Write(outMsg.ToString());
            outMsg.Clear();
        }
    }
    Console.WriteLine(outMsg.ToString());
    Console.ForegroundColor = ConsoleColor.Yellow;
    Console.Write("\nUser: (bye or nothing to exit chat)");
    helpTopic = Console.ReadLine() ?? string.Empty;
    if (helpTopic.Equals("bye", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }
}
Console.ForegroundColor = ConsoleColor.White;
```
