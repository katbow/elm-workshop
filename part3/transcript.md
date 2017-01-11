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
we're going to do it. Each of these results has a data field. When you click on
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

You can see we've got this initial model. You always need to specify an initial
model in order to kick things off. This it the thing that we will be updating
with our function, this initial value. We've got a bunch of hardcoded results.
Before we had one hardcoded result. We've expanded that, we're really moving up
in the world, now we have several hardcoded results in a list. We've also got
this query thing. Currently that's set to `"tutorial"`, it's going to be,
someday, something that gets displayed in a little search box. It's going to,
by default, search for tutorial. But for now, we're not going to use it yet,
we're just going to focus on the results.

Each result has it's own id. It's has a name, and stars. We still have our
`elmHubHeader` thing. Note by the way, that I went ahead and pulled this up to
the top. This used to be in a `let`, but I said "Hey, this doesn't actually
have any dependencies, so might as well just put it up as a top level value
instead of making a let for that." That's kind of usually good practice. If you
don't have any particular reason to have a let, if you have a choice between
putting something in a let and pulling it out at the top level, pulling it out
at the top level like this makes it clear what the dependencies are. This
doesn't have a dependency on anything else in my program, it's just totally
stand alone. The only scope that it needs is the top level scope itself. That's
generally the preferred way to go with that.

We've got `view`. `view` now takes `model` as an argument. Initially when we
call `view`  it's going to get this thing, our initial model. On subsequent
runs, after the user clicks and does things, it's going to be a potentially
different model, so we need to take into account this model right here. We
don't want to work in terms of that initial model, or we're going to be working
with stale application state. It's much better to just nly deal with this model
argument that we're getting passed in here. So we've got class content,
`elmHubHeader`, all the same as before. Now our `ul` that has the results class
is using `List.map`. What is it doing with `List.map`? It's looking at
model.results, this big old list of things right here and what it's going to do
is it's going to map `viewSearchResults` over that. It's going to go each of
those results, those individual search results, and it's going to call this
function on it. This is where we're going to do the bulk of heavy lifting on
our view logic. Each individual thing has the class `"star-count"`, just like
before. We've got that `text (toString result.stars)`. All good so far. We've
got our `a` with our `href`, just like before. I did that little
`target "_blank"` so when you click on the links, it opens in a new tab. It's
just a "nice-to-have" for our users. `text result.name`, all the same as
before. But *now* we've got this new button. And this button, it just says "X".
It's got some styling called `hide-result`. What we want to do, we want to make
it so that when the user clicks on this, this sends one of those
`"DELETE_BY_ID"` messages that we were talking about before, which will then
get picked up by `update`.

So `update` is going to look at this message and it's going to say "Hey, did I
get a 'DELETE_BY_ID' message? Cool. If so, look at its `data` field to find out
which `id` the user clicked on. Then remove that from our model's result list
using `List.filter`." Important to note that these two need to coordinate.
`onClick` cannot just give the same exact message every single time. It needs
to give a message that sends the particular id of the search result that we're
rendering right now. That way when `update` gets it, it will get the right `id`
and it knows which one to remove from the list.

Any questions on that? Yeah?

"I just noticed `viewSearchResult` is defined lower, you know after."

"Ahh, yes"

"So, do you want to explain?"

So `viewSearchResult` is actually being defined after it's being used inside
`view`. In general, any of these top level definitions, you can order them
pretty much however you want. This is getting into some stuff we haven't really
gotten into yet, but the short answer is that because of Elm's guarantees
around immutability and side effects and things, it's always safe to do this,
because execution order is not determined by order in the file, but rather by,
essentially, the Elm architecture. It's going to call these things in certain
orders. You actually can't really write something up here that would cause some
surprising thing to happen that matters. So you can pretty much reorder anytime
you want, however you want.

Let's go ahead and work through this. Here's what we've got so far. We've got
each of these things as a rendering from the inital model. I've got these "X"s
here which do nothing! I'm clicking, you probably can't tell that I'm clicking,
but trust me I'm clicking and it's not doing anything. We're rendering each of
them with that `List.map`, which is cool. We've got our header which is great.
But we don't actually have any interaction yet. So let's hook it up. First
thing we want is some kind of `onClick` handler that's going to send one of
these `"DELETE_BY_ID"` messages. We're going to add that right after the
class. `onClick`. Then we're going to put the actual message in here:
`{ operation = "DELETE_BY_ID", data = result.id }` For data, we're saying we
want the particular id of the exact result we're looking at right now. So
that's just going to be `result.id`. Okay, save that. Now, that's only one half
of the equation. We need both to send that particular value, but also to make
it so that update receives that value. Right now we're just sending it into the
ether and it's coming through here, but update just returns the same model
everytime anyways. So let's make it do something a little more useful.

```elm
if msg.operation == "DELETE_BY_ID" then
```

`if` this is true, we want to do something more useful.

```elm
if msg.operation == "DELETE_BY_ID" then
  model
else
  model
```

`else` we're going to just leave the model alone.

Specifically what we want to do is return a new model without the given `id`
present anymore. So how are we going to do that?

We'll start with this record update. So we want to say, "Leave everything in
the model alone, except for the results field."

```elm
if msg.operation == "DELETE_BY_ID" then
  { model | }
else
  model
```

So we're going to leave `query` alone and just mess with `results`.


```elm
if msg.operation == "DELETE_BY_ID" then
  { model | results = model.results }
else
  model
```

But, we want to transform this. Right now we're saying "Take the model, and set
it's results equal to the same results we had before and give me a model that
looks like that." This right here is elaborately worthless. What we want to do
instead is transform this somehow, so actually return something new and different.

```elm
if msg.operation == "DELETE_BY_ID" then
  { model | results = List.filter (\result -> ) model.results }
else
  model
```

We need to give `List.filter` some sort of function. I'm going to use an
anonymous function style here `(\)`. What is a given element in here? What
does one of these results look like? It's one of these things, one of these
nested records with `id`, `name`, and `stars`

```elm
{ id = 1
, name = "TheSeamau5/elm-checkerboardgrid-tutorial"
, stars = 66
}
```

I'm going to call that result so that's the argument to my anonymous function.
When I get one of these results how do I decide if it's keepable or not? How do
I decide if this particular result should stay in the list? Well, one easy
mistake to make is:

```elm
if msg.operation == "DELETE_BY_ID" then
  { model | results = List.filter (\result -> result.id == msg.data) model.results }
else
  model
```

Some of you may be smiling because you did this. If you do that, anyone know
what I'm going to get when I click on one of these? "Only that one." Yeah, it
will delete *everything* else, it will only *keep* that one. Not quite what we
want, small bug. Elm's compiler is pretty awesome, but it cannot save you from
that error. Business logic errors are always fair game. I think that's just
always going to be true in programming. If I change it to
`result.id /= msg.data`, I saw somebody in the chat mention `/=` is same as
`!=` in JavaScript which is totally true. We briefly mentioned that. That's
one way to write this. Now when I click on this, it does the right thing! Sweet.
Other ways we could write this if we wanted to get fancy is we could have also
left `==` alone and just written `not result.id == msg.data`.
This will do exactly the same thing. I would personally choose to use the `/=`
though just because it's a little more concise. There was another question
earlier about the type of message and are there any restrictions on that? No,
you can make *any* kind of message you want to come up with. We could make it a
string, we could make it an int, we've chosen to make it a record in this case,
could also be a list, who knows. Typically though, there's one convention for
how to do these and everybody does that. That's not quite what we're doing
right now though, we'll learn that in part 5. But basically, if you want to,
you can use pretty much whatever you want for your messages. One guiding
principle though is it's recommended that you do not put functions in your
message, like unevaluated functions, and also that you don't put unevaluated
functions in your model. It will let you; it will compile and it'll work, but
architecturally speaking, that's not something you want to be doing. It's just
a rule of thumb.  
