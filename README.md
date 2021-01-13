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


# Algorithms
## All permutations of bool array
```kotlin
fun main() {
  CrazyRunGeneratorService.allPermutationsOf(listOf(true, true, false, false, false))
    .map { println(it) }
}

object CrazyRunGeneratorService {

  private var crazyRunDebugTrack = 0

  fun allPermutationsOf(list: List<Boolean>): List<List<Boolean>> {
    val result = mutableListOf<List<Boolean>>()
    var v = 0b00000
    var current = 0b00001
    list
      .sortedDescending()
      .forEachIndexed { index, b ->
        if (b) {
          v = v or current
        }
        current *= 2
    }
    var w = v;

    var currentString = w.toString(2)

    while (currentString.length <= list.size) {
      result.add(currentString.padStart(list.size, '0').map { it == '1' })
      val t = (v or (v - 1)) + 1
      w = t or ((((t and -t) / (v and -v)) /2) - 1)
      v = w;
      // w is next lexicographically
      currentString = w.toString(2)

    }

    return result;
  }
}
```

outputs
```
[false, false, false, true, true]
[false, false, true, false, true]
[false, false, true, true, false]
[false, true, false, false, true]
[false, true, false, true, false]
[false, true, true, false, false]
[true, false, false, false, true]
[true, false, false, true, false]
[true, false, true, false, false]
[true, true, false, false, false]
```
