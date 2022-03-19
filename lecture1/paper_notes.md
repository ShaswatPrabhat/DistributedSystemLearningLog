# Map Reduce Paper

Map : (Key1, value1) => (Key2, value2)

Reduce : (Key2,list(value2)) => list(value2)

Map invocation distributed across multiplle machines
-> Partition input data in M Splits
-> Execute in parallel
-> Reduce partitions intermediate keyspace (Key2) into R spaces by using partition functions like hash(key) mod R
-> 

## Aim

* To create parallelizable tasks.
* To Abstract the above.
* Key and Reduce are pure functions to be parallelizable.
* To make distributed transactions more approachable.
* Parallelize computation and deal with failures easily.
* Use Re-exeuction as a way to add fault tolerance.

## Examples

Number of occurences of each word in a large document

```python
    map(String key, String value):
    // key: document name
    // value: document contents
    for each word w in value:
      EmitIntermediate(w, "1");
```

```python
    reduce(String key, Iterator values):
    // key: a word
    // values: a list of counts
    int result = 0;
    for each v in values:
      result += ParseInt(v);
    Emit(AsString(result));
```

* Distributed Grep: The map function emits a line if it matches a supplied pattern. The reduce function is an identity function that just copies the supplied intermediate data to the output.

* Count of URL Access Frequency: The map function processes logs of web page requests and outputs ⟨URL, 1⟩. The reduce function adds together all values for the same URL and emits a ⟨URL, total count⟩ pair.

## Typical exdcution overview

* Split input into M pieces.
* Run MapReduce instances in a cluster of machines.
* One of these is the Master
* Master picks up each map task (M in number) and each reduce task (R in number) and assigns them to workers.
* Worker given Map task -> Reads content of the input split -> parses key/value pair from them -> passes to Map function -> sends key/value buffered to memory.
* buffered value in memory is written and then paritioned -> Master gets the memory location of these partitons
* Reduce worker gets the localtion -> Uses RPC to read this data from memory location of map workers -> Gets the Key/Value -> Sort the Keys ( because same keys map to the same reduce task)-> If intermediate data is too large use **external sort**
* Reduce worker iterates over this sorted data-> for each key/set of values passes to Reduce function -> Value is appended to final output log
* O/P of Map task -> Stored in local disk of Map tasks
* O/P of Reduce tasks-> Stored in global storage 


## Granularity of tasks

* Divide Map in M parts
* Reduce into R parts
  