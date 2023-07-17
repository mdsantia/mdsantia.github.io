---
layout: single
title: "A fun multi-lingual concurrency experiment"
date: 2023-03-04
mathjax: true
---

The other day I was writing some code for a CI pipeline that needed to fetch release notes from 
several different git repos and collate them into one large set of release notes for all the different
services. My first instinct was to write a little python helper function and use that in the CI pipeline. 
On the first iteration I just looped over the URLs for the different services and did a GET request, then I 
collected the results and did some formatting on them. Even though there were only a few GET requests I noticed this 
took a few seconds to run. Then I started thinking about concurrency and implemented something in Go for fun because
I've been obsessed with Go recently. Then I implemented the same thing in Python and good ole Bash.

This post will mostly just go over these implementations, do a little explanation and mainly just point out that
the overall code is nearly identical for these three languages. Then at the end we will see which one is the fastest.

In this example I am using a placeholder API to grab titles of TODO items. I add a 500ms delay to simulate a real endpoint that is a bit slow.

All the code here can be found on [Github](https://github.com/jellis18/concurrent-get-requests-example).

The overall structure is the same for all three. 
1. Create function to fetch a single todo given its ID
2. Concurrently fetch all TODOs using this function
3. Process TODOs and output a single list of TODOs in markdown format

## Go Version
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"os"
	"strconv"
	"sync"
	"time"
)

const (
	todoServer = "https://jsonplaceholder.typicode.com/todos"
)

func getTodo(index int) (string, error) {
	time.Sleep(500 * time.Millisecond)
	resp, err := http.Get(fmt.Sprintf("%s/%d", todoServer, index))
	if err != nil {
		return "", err
	}

	if resp.StatusCode != http.StatusOK {
		return "", fmt.Errorf("Error getting todo %d: %s", index, resp.Status)
	}
	defer resp.Body.Close()

	var todo struct {
		Title string `json:"title"`
	}

	if err := json.NewDecoder(resp.Body).Decode(&todo); err != nil {
		return "", err
	}

	return todo.Title, nil
}

func main() {
	numTodos, err := strconv.Atoi(os.Args[1])
	if err != nil {
		log.Fatalf("Error parsing number of todos: %v", err)
	}

	var wg sync.WaitGroup
	todos := make([]string, numTodos)

	for i := 0; i < numTodos; i++ {
		wg.Add(1)
		go func(j int) {
			defer wg.Done()
			todo, err := getTodo(j + 1)
			if err != nil {
				log.Printf("Error getting todo %d: %v", j+1, err)
			}
			todos[j] = todo
		}(i)
	}
	wg.Wait()

	fmt.Println("# Todos")
	for _, todo := range todos {
		if todo == "" {
			continue
		}
		fmt.Printf("* %s\n", todo)
	}
}
```
In the Go version I used waitgroups instead of channels since I want the output to be ordered and it is just easier with waitgroups. 
In the main function we loop over the TODO IDs and make an inline goroutine to get the todo and add it to the TODOs array. Notice that we make
the array beforehand and are not appending to a slice, this would lead to race conditions. We then wait for all goroutines to finish by using `wg.Wait()`, lastly we loop over the TODOs that we have pulled down and make a markdown list with a title. Overall its very simple and concise. As usual, most of the lines are error checking.

## Python Version
```python
import asyncio
import logging
import sys
from typing import Optional

import httpx

TODO_SERVER = "https://jsonplaceholder.typicode.com/todos"

logger = logging.getLogger(__name__)


async def get_todos(client: httpx.AsyncClient, index: int) -> Optional[str]:
    try:
        await asyncio.sleep(0.5)
        response = await client.get(f"{TODO_SERVER}/{index}")
        response.raise_for_status()
    except httpx.HTTPStatusError as exc:
        logger.warning(f"Failed to get todo {index}: {exc}")
        return
    return response.json()["title"]


async def main(num_todos: int) -> None:
    async with httpx.AsyncClient() as client:
        tasks = [get_todos(client, index + 1) for index in range(num_todos)]
        results = await asyncio.gather(*tasks)

    print("# Todos")
    for todo in results:
        if todo is None:
            continue
        print(f"* {todo}")


if __name__ == "__main__":
    asyncio.run(main(int(sys.argv[1])))
```
In the Python version I used the `asyncio` library to make the concurrent requests using `asynico.gather` and used the `httpx` library to make the asynchronous HTTP requests. Like the Go version we have a separate `get_todos` function that takes in a client and an index and returns the title of the TODO. In the main function we create a client and then create a list of tasks that we will run concurrently. We then use `asyncio.gather` to run all of the tasks and wait for them to finish. They will be gathered in the order that the list comprehension was created with. Lastly we then loop over the results and print out the TODOs in markdown format. Overall this is slightly shorter than the Go version. There are many ways to do nearly the same thing in Python, but this is probably the most popular way nowadays since async/await was added to the language.

## Bash Version
```bash
#!/bin/bash

TODO_SERVER="https://jsonplaceholder.typicode.com/todos"
NUM_TODOS=$1


get_todo(){
    sleep 0.5
    curl -s --fail $TODO_SERVER/$1 | jq -r '.title' > "/tmp/todo_$1.txt"
}

for ((i=1; i<=${NUM_TODOS}; i++));
do
    get_todo $i &
done

wait

echo "# Todos"
for ((i=1; i<=${NUM_TODOS}; i++));
do
    echo "* $(cat /tmp/todo_${i}.txt)" && rm "/tmp/todo_${i}.txt"
done
```
This one was the most fun to write. Even though I use bash everyday in CI/CD tasks, I usually don't write full scripts or try to do much concurrently. To mirror the structure of the other implementations I created a function to get a single TODO using curl and jq and then looped over the TODO IDs and called the function in the background. The big difference here is that since by running in the background we are using separate processes and it is hard to share data between them. To get around this I used a temporary file to store the TODO title with a filename that was the TODO ID.

I then used `wait` to wait for all the background processes to finish. Lastly I looped over the TODO IDs again and printed out, read in the content of the temporary files and output the TODOs in markdown format. This implementation is the shortest and most concise, but, as we will see in the next section, it is also the slowest.

## Benchmarking
So I didn't do any fancy benchmarking, I ran each version with 200 TODOs and ran them 10 times and took the average. The results are below:
<table>
  <tr>
	<th>Language</th>
	<th>Average Time (s)</th>
	<th>Uncertainty (ms)</th>
  </tr>
  <tr>
	<td>Go</td>
	<td>0.71</td>
	<td>26</td>
  </tr>
  <tr>
	<td>Python</td>
	<td>1.42</td>
	<td>121</td>
  </tr>
  <tr>
	<td>Bash</td>
	<td>3.84</td>
	<td>540</td>
  </tr>
</table>
So as we can see the results are probably what you would expect. Go is fastest, Python is second and Bash is last; however, remember that all of these would have taken ~100 seconds if they were done sequentially. So even though Bash is the slowest, it is still 26x faster than the sequential version. Bash is slowest because it us spawning a new process for each TODO and doing extra IO with the temporary files. For Python vs Go, I'm sure there is a full reason why it is about 2x slower than the Go version but I think overall it is just because the Go version is compiled and the Python version is interpreted. Furthermore Go was specifically build with concurrency in mind and it is more of an add-on to Python.

## Conclusion
So this was a strange one but I had fun playing around with it and hopefully you will dig into it a bit more. I'm planning on more posts using Go as I am really starting to like it and I think this post shows some of its power and simplicity. I'm also planning on doing some more posts on Python and Bash, so stay tuned. If you have any questions or comments feel free to reach out to me on Twitter or leave a comment below. Thanks for reading! (These last two sentences were completely generated with Copilot but it sounds right, so there it is.)

