# Transcript [![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/rtfeldman/elm-workshop/issues)

// part3 starts at 43:44 in hr1
// this section begins in hr2 and goes until ~18:30 into the vid

Finally, we have update and this is how things get changed over time.
When the user is interacting, ultimately, updates job is to say, "I am going
to take us from the current model to a new model, based on whatever the user's
interaction was." Update, model and view form the baseline for all interaction.
The user does something, we update the model, and then view gets run once more
with the new model in order to produce some new Html, and the Elm Runtime takes
care of the rest.

Let's look at an example of how this would actually look in practice. Pretend
we have this potential, hypothetical feature. We have some search results and
we want to say "I want to have a max results, which is to say, the maximum
number of results to show, and a show more button that will just keep adding
to that. It will show more and more things." It's a pretty basic interaction.
\The first thing we want to do for this, is we want to represent the idea of
the user requesting to see more as data. We're going to make a value, or a
record in this case, that's going to be what we're going to call our message.

```elm
{ operation = "SHOW_MORE"
, data = 10
}
```

This message is what's going to get sent to the update function to tell it,
"Hey, you need to change the model in a certain way." When the user clicks that
show more button, we're going to make one of these exist, and we're going to
send it to the update function. Here's what our update function might look like:

```elm
update msg model =
  if msg.operation == "SHOW_MORE" then
    { maxResults = model.maxResults + msg.data }
  else
    model
```

`update` takes two arguments. The first argument is always the message, and the
second argument is the model: the current model, the current state of the
entire application. Then it's going to look at the message that you requested,
and return a new model based on what that message is telling the update to do.
In this case it's saying that "I see that this is a `SHOW_MORE` operation," and
it's got some extra data in here, it's got the number 10 which let's say is the
number of extra results to show. We're going to say "Cool, I'm going to return
a new model, which is basically a new record that's got
`maxResults = model.maxResults + msg.data`". This would increment this by 10
everytime. Notice that we're not mutating anything in place. We're not saying
"Go change my model right now!" What we're doing is we're just saying, you gave
me this message, you gave me this current model, here's your new model. This is
the new model that I want you to use as the new source of truth for our
application state. `else`, if we didn't get this operation, this `SHOW_MORE`
operation, if we got some other message, any other message, just return the
existing model. It's saying I have no changes here, just use the same model as
you got before. Nothing to see here, move along.

There's a slight issue here, which is that this exact approach would not scale
terribly well. This is assuming that our model has exactly one field in it,
`maxResults`. Now we saw earlier that the model we're going to be working with
has things like query and results and stuff like that, so what are we going to
do? Are we going to put query = query, results = results? That would be really
annoying. Fortunately we don't have to do that, because Elm has **record update
syntax**.

```elm
update msg model =
  if msg.operation == "SHOW_MORE" then
    { model | maxResults = model.maxResults + msg.data }
  else
    model
```

It's just like what we had before, except we've added this `model |` (pipe) bit
in here. `model |` basically says, "Start with our model and then I want to
return a record that is exactly the same as that, except *this* is different".
So, it's exactly the same as the model, except that `maxResults` has a different
value, and here's the value I want for `maxResults`. You can do multiple updates
in the same thing, so you can also add `, foo = bar` or whatever other stuff in
there. But the relevant part to know about this, is again, this does not
actually change the model. Again, everything is immutable, there's no actually
updating in place, this is just giving us a brand new model. Another thing to
note is that this is actually pretty efficient. It's not literally cloning the
entire model, but conceptually you can think of it that way, but from a
performance perspective, it's not actually going through every single field and
making a duplicate of it. It's using persistent data structures under the hood
that do that more efficiently. If you have a model with a thousand fields in it,
and you do this, you're not going to get two thousand worth of fields of stuff
in memory. You're actually just going to get a very large shared structure with
one little change for `maxResults` being different. Those two will coexist, but
the remaining 999 things will be shared between those two data structures. So
we don't have to worry about that, this is actually a fast operation.

OK, so, we got our message, and `msg.operation == "SHOW_MORE"`, cool return the
same model we had before but with this one change. Great, that's our new model.
Essentially, that sort of completes the cycle. We get a message in somehow from
the runtime, it goes into `update`, we run the update function and it says "Got
it, I've got my message that you just gave me." You've got the previous model,
which also gets passed in. It returns a new model. That model gets passed to
view. View returns some Html, and Elm Runtime says "Cool, I'm going to make that
Html go on the screen." All of this put together gives us the idea that all we
need to do in order to make interaction happen is describe under what
circumstances we want messages to get sent to update like this. We're going to be
finding that messages are sort of our *currency* for dealing with interaction as
well as some other things. When we think about how do we want the application
state to change, we're always going to be describing how we want the application
state to change by using these messages.

So, `onClick`: this is a function that's built in to Elm's Html thing. We saw
this message:

```elm
{ operation = "SHOW_MORE", data = 10 }
```

Here's how we set up a click handler:

```elm
onClick { operation = "SHOW_MORE", data = 10 }
```
 You just add `onClick` onto the front. You say onClick, and you say the message.
 What that's going to do is that's going to say whenever the user clicks on this
 element, send that message, that exact message right there, to update along with
 the current model. It's going to keep that whole cycle going. This is just an
 attribute that we can put in our list of attributes.

```elm
button
  [ onClick { operation = "SHOW_MORE", data = 10 } ]
  [ text "Show More"]
```

And that's it. That's the entirety of the wiring necessary to make all of that
happen. We're not going to say `addEventListener`, we're not going to modify
this button, we don't even know that this button exists this is just our view
code saying "Hey, when you render this button, I want it to have this behaviour.
Whenever the user clicks, send this message right here to update." The Elm
Runtime will automatically take care of adding and removing event listeners.
You don't have to manage that yourself, it just happens for you behind the
scenes. But the relevant part is that this is how we're going to introduce
interaction to our application on a basic level.

So, we got our view, it's returning some Html, we got our model, which looks
something like this:

```elm
{ query = "tutorial"
, results = []
}
```

And we're going to introduce in the next exercise, `"DELETE_BY_ID"` as our first
message. The first example, we just had some Html that we were putting on the
screen, the second example we introduced a thing which we called model but we're
not really using it in this way, and now we're going to introduce `update` and
`msg`. Here's how we're going to do this. We're going to introduce a feature
where we're rendering a series of these repositories, and we're going to have
a little `X` button next to each one. The user can click that button to remove
it from the rest. Just hide it, delete it, get it out of there. This is how
we're going to do it. Each of these results has a data field. WHen you click on
it, it's going to send a message that looks like this to update:

```elm
{ operation = "DELETE_BY_ID"
, data = 2
}
```

where data is whatever particular `id` of that result is. Then inside update,
we're going to write some logic that receives one of this messages and uses
`List.filter` to take that particular result, with that particular id out of
the list. That will generate a new model, which will get passed to view, which
will give us some new Html. From the end user's perspective they're going to
think I clicked that "X", that thing went away, I'm happy.

We have the same pattern as before. We've got some exercises to go through,
we've got some TODOs. Let's take a look at those.
