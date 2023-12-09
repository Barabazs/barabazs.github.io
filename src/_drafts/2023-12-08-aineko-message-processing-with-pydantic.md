---
layout: post
title:  Consuming and Producing Aineko messages with Pydantic models
author: "Barabazs"
date:   2023-12-08 12:00:00 +0200
categories: posts
tags: [aineko, pydantic]
published: false
---
## Introduction
A common pattern across [Aineko] nodes is to consume and produce messages. Messages are implemented as dictionaries, and are passed around between nodes using datasets.
This offers a lot of flexibility, but all that flexibility can come at a cost when it comes to complex pipelines. The more datasets you have, the harder it is to keep track of the messages that are being passed around.

I'll describe my own approach to this problem, which is to use [Pydantic] models to define the messages. Pydantic is a library that allows you to define data models with type annotations, validation, and parsing out of the box.


## How to use them
My personal approach is to define a model for each dataset that is used in the pipeline. In most circumstances a dataset will only be used for a specific message type. 
More complex pipelines might use a single dataset for multiple message types, but even these can be handled with a single model or a model hierarchy.

I store these models in a separate file, and import them in the modules that need them.
The project structure looks roughly like this:
```
...  
├── nodes
│   ├── node1.py 
│   ├── node2.py
├── models
│   ├── messages.py
├── ...
```

Defining a model is simple. Let's imagine we have a dataset called `simple_event`, which is used to pass around simple event messages. The original message has the following structure:

```json title="simple_event message"
{
    "score": 0.8,
    "event_source": "node1",
    "detected_at": 1623345600,
    "payload": {
        "foo": "bar"
    }
}
```

We can define a model for this message like this:

```python title="messages.py"
# path: messages.py
from pydantic import BaseModel

class SimpleEventMessage(BaseModel):
    """Message model for the Simple Event messages."""

    score: float
    event_source: str
    detected_at: int
    payload: dict

```

Now let's see how we can use this model in our nodes. We'll be producing a message in node1, and consuming it in node2.


```python
# path: node1.py
from aineko.internals.node import AbstractNode

from models.messages import SimpleEventMessage

class Node1(AbstractNode):

    def _execute(self, params: Optional[dict] = None) -> None:
        # ... Code that does something ...
        message = SimpleEventMessage(
            score=0.8,
            event_source="node1",
            detected_at=time.time(),
            payload={"foo": "bar"},
        )
        
        self.producers["simple_event"].produce(message.dict())

```

```python
# path: node2.py
from aineko.internals.node import AbstractNode
from pydantic import ValidationError

from models.messages import SimpleEventMessage

class Node2(AbstractNode):

    def _execute(self, params: Optional[dict] = None) -> None:
        message = self.consumers["simple_event"].consume()
        
        if message is None:
            return None

        try:
            event_msg = SimpleEventMessage(**message)
        except ValidationError as e:  
            self.log(
                f"Error parsing JobCompletedMessage: {str(e)}",
                level="error",
            )
            self.log(f"Message: {message}", level="error")
            return None

        # Now we can use the message as a regular python object
        if event_msg.score > 0.5:
            ...

```

## Conclusion
Using pure dictionaries for messages offers an unopinionated approach, but it has some drawbacks:
- No type safety
- No validation
- No IDE completion
- Less suitable for complex messages (think nested dictionaries, optional fields, etc.)

Pydantic models solve all these problems, and are very easy to use.

## Notes
- The consume's return type is `Optional[dict]`, so we need to unpack it before passing it to the model.
- The `produce` method expects a `dict`, so we need to convert the model to a dictionary before producing it.
- The `**` operator is used to unpack the dictionary into keyword arguments, which is a correct and efficient way to instantiate a Pydantic model.
- The Aineko version used in this example is 0.2.5, which is the latest version at the time of writing this article.
- I'm an early engineer at [Aineko], but the opinions expressed in this article are my own.


[Aineko]: https://www.aineko.dev/
[Pydantic]: https://docs.pydantic.dev/latest/