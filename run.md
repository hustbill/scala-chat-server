#Run it

##Download and build Akka
1. Check out Akka from http://github.com/akka/akka  
2. Set ‘AKKA_HOME’ environment variable to the root of the Akka distribution.  
3. Open up a shell and step into the Akka distribution root folder.  
4. Build Akka by invoking:  
% sbt update  
% sbt dist  

##Run a sample chat session  
1. Fire up two shells. For each of them:  
• Step down into to the root of the Akka distribution.  
• Set ‘export AKKA_HOME=<root of distribution>.  
• Run ‘sbt console’ to start up a REPL (interpreter).  
2. In the first REPL you get execute:  
```scala
import sample.chat._  
import akka.actor.Actor._  
val chatService = actorOf[ChatService].start()  
```
3. In the second REPL you get execute:
```scala
import sample.chat._  
ClientRunner.run  
```
4. See the chat simulation run.  
5. Run it again to see full speed after first initialization.  
6. In the client REPL, or in a new REPL, you can also create your own client  
```scala
import sample.chat._
val myClient = new ChatClient("<your name>")
myClient.login
myClient.post("Can I join?")
println("CHAT LOG:\n\t" + myClient.chatLog.log.mkString("\n\t"))
```  

That’s it. Have fun.  
