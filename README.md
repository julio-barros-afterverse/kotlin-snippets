# Kotlin Snippets

## Actors
### Deadlock
Rendezvous (capacity 0) actors will deadlock here:
```kotlin
suspend fun main() {
  var g: suspend () -> Unit = {}

  val c = GlobalScope.actor<Int>(capacity = 2) {
    for (message in channel) {
      println("received $message")
      delay(20)
      if (message == 2) {
        g()
      }
      println("finished receive of $message")
    }
  }

  g = { c.send(4); println("sent 4") }

  delay(100)
  c.send(1)
  println("sent 1")
  c.send(2)
  println("sent 2")
  c.send(3)
  println("sent 3")
  c.close()
}
```

### Actor suspension
Actors `send` suspend if called while actor is processing a message
```kotlin
suspend fun main() {
  val c = GlobalScope.actor<Int> {
    for (message in channel) {
      println("received $message")
      delay(20)
      println("finished receive of $message")
    }
  }

  delay(100)
  c.send(1)
  println("sent 1")
  c.send(2)
  println("sent 2")
  c.send(3)
  println("sent 3")
  c.close()
}
```
outputs
```
sent 1
received 1
finished receive of 1
sent 2
received 2
finished receive of 2
sent 3
received 3
```

## Channels
### BroadcastChannel
When conflated, broadcast channel stops processing and ignores messages when closed
```kotlin
suspend fun main() {
  val c = BroadcastChannel<Int>(Channel.CONFLATED)

  val consumer = GlobalScope.launch {
    c.consumeEach {
      delay(20)
      println("received $it")
    }
  }
  delay(100)
  c.send(1)
  println("sent 1")
  c.send(2)
  println("sent 2")
  c.close()
}
```
outputs
``` 
sent 1
sent 2
```
