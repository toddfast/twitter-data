Originally published May 12, 2009. This is version 0.3, published May 19, 2009.

Introduction
------------

> *In 140 chars: Twitter Data lets people embed bits of data in their
> tweets so that computers can read the data and do cool stuff
> \#twitterdata*

Twitter Data is a simple, open, semi-structured format for embedding
machine-readable, yet human-friendly, data in
[Twitter](http://twitter.com) messages. This data can then be
transmitted, received, and interpreted in real time by powerful new
kinds of applications built on the Twitter platform. Here is an example
Twitter Data message:

````
I love the \#twitterdata proposal! $vote +1
````

The part with the dollar sign, `$vote +1`, is a piece of data embedded
using the Twitter Data format.

When an application knows how to read this data, it can then display it
to users or send it to other applications to make Twitter an even more
interesting and useful platform. You can see an example of this in the
voting widget over there in the right-hand column of this page.

### How it works

1.  A user (or her Twitter client) sends a Twitter message that includes
    embedded Twitter Data, like `Watch out, just saw a #police car near  San Francisco Airport $lat 37.612804 $long -122.381687`
2.  This message is distributed like any other Twitter message to the
    user's followers
3.  The followers' Twitter clients listen for messages with embedded
    Twitter Data and extract data they understand. This data is
    presented to users, recorded, or otherwise consumed in some useful
    way. In this example, the Twitter client could show a map with a pin
    for the police car's location.
4.  Data mining applications extract Twitter Data from messages using
    the Twitter platform (for example, using search or the public
    timeline) and then process it, present it, or otherwise consume it
    in some way useful to humans or other applications. In this example,
    it would be possible to build a map showing police car locations in
    real time based on user reports via Twitter.

### Why?

Twitter is an amazing, constantly changing stream of information, most
of which is unstructured, with useful data locked up in the
near-infinite complexity of human language. The inability to easily
extract this data limits the usefulness of that information,
particularly for applications that want to interact with humans on
Twitter in interesting ways. For example, use cases such as attaching
metadata about message senders or distributing shared data within a
social graph are difficult to solve without using some structured data
format.

When data can be formally embedded in Twitter messages, we see enormous
new opportunities for data mining, social data distribution, and
intelligent application behavior unlimited by lack of natural language
processing technology. Best of all, the richness of this data is
available to everyone, and can be sent and consumed by both humans and
machines.

### Goals

By proposing an embedded data format, our goal is not to turn Twitter
into a mere transport layer for machine-readable data, but instead to
allow semi-structured data to be mixed fluidly with normal message
content. To these ends, we have chosen a syntax that conceptually
resembles the use of Twitter hashtags, albeit with different syntax and
semantics, and which allows humans to interact with data in a reasonably
normal way.

We are publishing this proposal to invite peer review, spark useful
debate, and gather consensus. In short, we think this is a good idea and
hope you will too, but we need your help to vet it.

### Terms

Applications
:   Programs that send and/or receive messages that may contain Twitter
    Data tuples
Message frame
:   A single message containing Twitter Data tuples, normally with a max
    length of 140 characters on Twitter. Message frames may be embedded
    in other messages frames, for example in the case of retweeting.
Tuple
:   A single pair of name and value used to send a single datum
Twitter Data
:   A distinct dimension of machine-readable data that can be embedded
    in Twitter or Twitter-like messages

Syntax
------

### Tuples

Data is structured as simple name-value pairs of strings, called
*tuples*, embedded in normal Twitter messages. The simplest form is the
following:

````
$name1 value1
````

This example shows a single tuple, with the name "name1" and the value
"value1".

Names and values are separated by one or more spaces. These spaces are
not considered part of the name or the value. Values start with the
first non-space character following the termination of the name with a
space. Therefore, this example is equivalent to the one above despite
the number of spaces between name and value:

````
$name1        value1
````

Spaces within values are literal and unescaped:

````
$name1 some long value
````

Multiple tuples can appear in a single message frame, where the value of
the previous tuple is terminated by the start of a new tuple name
preceeded by a space:

````
$name1 value1 $name2 value2
````

If a tuple appears at the end of the message, then the value terminator
is implicit. If the tuple is embedded in the message with non-data text,
the last tuple's value before the text begins must be terminated by a
dollar-sign delimiter following the last character of the last value,
immediately followed by a space:

````
some text $name1 value1 $name2 value2$ some more text
````

Multiple groups of tuples are allowed, as long as they are each properly
terminated:

````
some text $name1 value1 $name2 value2$ some more text $name3 value3
````

If a tuple value contains the literal dollar-sign character ("\$"), the
literal character must be escaped by doubling the character to "\$\$".
If non-data message text contains the literal dollar-sign character, we
advise doubling the value where it may otherwise be confused with the
tuple syntax.

#### More syntax details

Tuple names begin with a dollar sign ("`$`") delimiter and are terminated
by the first space or non-alphanumeric character with the exceptions of
underscore ("`_`") or as noted below. Tuple names must begin with an
ASCII letter or underscore, and by convention, use camel case with the
first letter lowercased. This convention helps preclude namespace
collision with existing use of the dollar sign in normal messages when
writing either dollar amounts or stock symbols. [TBD: How does character
encoding in Twitter affect this? Is it reasonable for UTF-8 strings to
be used as tuple names?]

Tuple values are terminated by the start of a new name delimiter; a
single, value-trailing delimiter; or the end of the message frame.

Note, tuples without a value are considered invalid and should be
excluded from processing by applications. Specifically, there is no
formal representation of tuples with null values; if a tuple is present,
it must have a value, however trivial. Our reasoning for this
restriction is twofold: First, that such usage overlaps with the
semantics of Twitter hashtags, and second, that it limits the amount of
accidental interpretation by applications of non-Twitter Data message
content that uses the dollar sign notation in other ways

If leading spaces are required in a value, the value should typically be
quoted (with single or double quotes), or the space characters would be
escaped, for example with URL encoding ("%20"). However, quotes or other
encoded forms are considered part of the tuple value and interpretation
of them in relation to the value rests with the receiving applications.
This proposal does not specify a standard mechanism for quoting or
encoding values. [TBD: Why not?]

#### Tuple recency

The same tuple name may appear more than once in the same message frame.
If each tuple applies to a separate subject, then normal semantics
apply. If all tuples apply to the same subject, then the rightmost tuple
should be considered the more recent. Note that recency does not
necessarily imply that a tuple value is exclusive with a prior tuple
value. Depending on the semantics of the tuple and/or subject, multiple
tuple values over time may be additive.

For example, in the following message frame:

````
@toddfast $likes movies $likes Twitter
````

it seems obvious that the `$likes` tuples should be non-exclusive.
Compare to these examples, in which tuples should be exclusive because
they describe momentary states of the subject:

````
I love #twitterdata $vote +1$ but now I hate it $vote -1
````

````
@toddfast driving $mph 65$, oops just hit a wall $mph 0
````

The interpretation and semantics of any tuple values that appear in a
message frame or over time in a message stream is beyond the scope of
this proposal. We expect the semantics of commonly-defined tuples with
regard to exclusivity of multiple values to be addressed by community
convention.

#### Reserved tuple names

We expect tuple names to be used by the community following
community-developed conventions. However, this proposal does reserve all
un-namespaced tuple names of a single letter or underscore.
Specifically, the following un-namespaced tuple names are reserved and
should not be used by applications:

````
a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, 
y, z, _, foo, bar
````

#### Comments on syntax

Some may question the use of the dollar-sign ("\$") as a tuple
delimiter. One main driver for this choice is that there is a very
limited set of non-alphanumeric characters that are compatible with
Twitter's search engine. Namely, Twitter search treats most
non-alphanumeric characters in a special way that limits their
usefulness in conjunction with this proposal. Note, this could change if
Twitter works with the community to formalize use of an alternate
character.

Here are some of our justifications for using "$":

-   Similar syntax to PHP and other scripting languages
-   The use of the character in this way seems to be relatively unique
    on Twitter and thus is available for use for encoding data
-   Compatible with Twitter search, making it possible to use for
    partial searches with other characters in conjunction.
-   It is human readable and doesn't interfere too much with normal
    readability so that Twitter Data message frames can be useful to
    humans as well as machines.

There are some open issues with this choice, however:

-   The dollar-sign notation is used by PHP programmers to express short
    code snippets on Twitter. The namespace collision may or may not be
    an issue over the long term. As a precaution, we have reserved the
    \$foo and \$bar tuple names because they commonly appear in code
    snippets.
-   The dollar-sign notation is also used by Twitter users discussing
    stock symbols, for example \$MSFT, \$ORCL, \$JAVA. Usually the stock
    symbols are in upper case, which is part of the justification for
    Twitter Data to default to lowercase tuple names.

### Simple namespaces

Namespacing is a key capability that allows Twitter Data tuples to be
interpreted with strict, well-defined semantics. The namespace syntax is
the following:

````
$namespace>name value
````

Namespaces comprise the portion of the tuple name following the name
delimiter "$" and preceding the greater-than symbol (">") symbol. They
follow all syntax rules as described above, and thus may not contain
spaces or other non-alphanumeric characters. Only one namespace may
appear in a tuple name.

Our intent is that the presence of namespaces allow the community to
define more precise semantics for interpretation of tuples. It is beyond
the scope of this proposal to define the semantics or content of any
particular namespaces at this time.

However, this proposal does reserve all namespaces of a single letter or
underscore, in addition to the "td" namespace. Specifically, the
following namespaces are reserved and should not be used by
applications:

````
a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, 
y, z, _, td
````

### Message subjects

In many cases, it is useful for data in Twitter Data messages to be
interpreted relative to a general topic, or *subject*. A subject is a
person, object, message stream, or other concrete or abstract entity to
which one or more tuples in the message frame apply. As explained below,
subjects may be implicitly determined by applications, or explicitly
specified in the message.

More than one subject may appear in a message frame, but any given tuple
is governed by only a single subject, called its *dominant subject*. By
convention, tuples are interpreted within the context of their dominant
subject, if present. A dominant subject is the subject that is closest
to a tuple:

> *The nearest subject identifier to the left of a tuple and outside a tuple value is the dominant subject for that tuple*.

where a subject identifier is either implicit or explicit as described
in the sections below.

In message frames without a determinable subject, the subject is
considered undefined, and tuples may be freely interpreted by any
application that purports to understand them. However, there are no
semantic guarantees in this case, and misinterpretation is possible.
Applications should therefore be robust in handling and filtering
unexpected tuple values (which is true even in the case where a subject
is present).

While tuples in message frames without a subject are not by definition
meant to apply to the message sender, in practice they often do. We
understand that this is a very natural way to interpret subjectless
tuples, but until further feedback is available from the community we
prefer to leave this to applications' interpretation.

#### Implicit subjects

Subjects are often implicit in a message, particularly in messages sent
by humans. The rule for determining the implicit subject of a message
frame is straightforward:

> *The nearest implicit subject identifier to the left of a tuple and
> outside a tuple value is the dominant subject for that tuple*.

where an implicit subject identifier is one of the following:

-   A hashtag, for example, \#twitterdata
-   A person tag, for example, @toddfast

Therefore, in the following example, the implicit subject is
`#twitterdata`:

````
Hey @toddfast, I like the latest #twitterdata proposal $vote +1
````

Implicit subjects make it very easy for humans to embed data in messages
without getting bogged down in hard-to-remember and verbose formalisms.
Furthermore, it makes messages with embedded data easier to read for
humans, and reduces the number of characters in a message frame.

#### Explicit subjects

In some cases, it reduces ambiguity or is clearer to explicitly specify
a subject. Usually, it will be applications sending messages with
embedded data that will want to explicitly specify a subject in order to
define a formal communication channel.

Subjects are specified using a tuple of the following form, comprising
the reserved tuple name "\$s" and a URI value, called the *subject
name*:

````
$s subject_name
````

The `subject_name` may be either a URI as defined in RFC 3986, a Twitter
person tag, or a Twitter hashtag. We strongly discourage use of other
classes of values.

Explicit subjects follow the same dominant subject rule as implicit
subjects:

> *The nearest explicit subject identifier to the left of a tuple and outside a tuple value is the dominant subject for that tuple*.

Applications that wish to partition their message streams from other
applications are strongly encouraged to use subject tuples with URI
values, such as shortened URLs (for example, shortened via bit.ly), that
are unlikely to collide with other casual subject name use. Such URLs
also conveniently allow the application to link to itself in the
messages it sends.

By convention, messages that start with the reserved subject tuple
("$s") may be considered by applications as intended for machine-only
consumption. Applications that present Twitter feeds to users may elect
to filter these messages from presentation.

It is beyond the scope of this proposal to define conventions for common
subjects, such as @ tags as a class of subjects, or specific hashtags.
We will leave this task to the community.

Note, the use of subjects potentially reduces the amount of namespacing,
thus saving valuable characters, and allows messages to be filtered more
efficiently by message recipients. In this sense, subjects might be
considered a default namespace for all tuples within a message frame,
though this is not strictly true. Furthermore, subjects may be appealing
because they are more unique and terse compared to simple namespaces
because the space of subject values is syntactically larger than that of
simple namespaces. However, namespaces and subjects are completely
compatible and will generally be used in conjunction with one another.

Other Considerations
--------------------

### Retweeting

Special consideration is necessary when considering the semantics of
data embedded in retweeted messages. Ideally, retweeting should not
change the interpretation of retweeted message frames. The purpose of
retweeting is to propogate a message through a different graph of
receivers, and so the central question is whether that instance of a
message should be considered separate from the original instance, and
thus whether it should be interpreted independently from the original.
We will discuss this topic more thoroughly based on use cases for
retweeted data in a future version of this proposal.

The determination of a subject for a message is somewhat more subtle in
the case of retweeted messages. For messages that either implicitly or
explicitly specify a subject, retweeting has no effect on the
interpretation of the message frame. However, in the case of subjectless
messages, the act of retweeting normally inserts a subject with the
inclusion of a person tag of the original author.

As described above, a subjectless message frame does not automatically
or necessarily apply to the message sender. We understand that this is
an incongruity that may be addressed by either special casing the
processing of retweeted messages, or by simply acknowledging that
subjectless messages probably apply to message senders and thus the
semantics of subject determination are neatly compatible with
retweeting. We would appreciate community feedback before making a final
proposal.

### Role and function of hashtags

Use of hashtags in Twitter Data is largely unchanged. If hashtags appear
in tuple values, they may be interepreted as normal hashtags in the
normal way by humans and user agents, and have undefined semantics in
terms of this proposal. By contrast, hashtags may not appear as part of
tuple names.

Hashtags will often be use as subjects of messages, which is
semantically consistent with their intended purpose.

### Binary data

This proposal is not suited to use cases that require streaming, binary
data, or large amounts of data to be exchanged. In particular, rate
limits of messages on Twitter combined with the limited message size
make Twitter Data unsuitable for transferring significant amounts of
data at high speeds.

In general, we discourage applications from using binary data with
Twitter Data, largely because it harms interoperability and is
inconvenient for users using Twitter to send human-readable messages.
However, binary data is unavoidable in some cases, and may be less ugly
than the equivalent human-readable serialization. If binary data is
sent, it should be Base64 encoded.

### Data size

A single tuple may not span more than one message frame of 140
characters. However, multiple tuples may be included in a single message
frame, with the requirement that all included tuples fit collectively
within the message frame size limit (on Twitter, 140 characters). Note,
we may introduce a means of frame spanning in a later proposal.

This specification does not specify a way to determine if tuple values
are truncated at the end of a message frame because the requirement of
terminating the set of tuples with a delimiter is optional. Applications
that receive tuples must be aware of this restriction and ensure that
they check for invalid values. Applications that send tuples should
instead attempt to send additional message frames with tuples that would
otherwise be truncated.

### Atomicity and transactional semantics

The usefulness and novelty of Twitter Data rests on the ability for
multiple messages containing varying data to be delivered over time, in
what we call *message streams*. Message streams are a virtual
consolidation of multiple messages around specific criteria, for example
a person, subject, or a search query. Note that messages may appear in
multiple message streams, in differing order.

The order of appearance of tuples in a message stream is undefined with
respect to this proposal, though it is most common for them to be
ordered in time, with the latest message at the head of the stream as we
nominally see in Twitter.

Data in message frames are intended to be non-atomic and
non-transactional. That is, there is no mechanism to determine whether a
collection of tuples should be considered one atomic group. Said another
way, there is no mechanism to specify the semantics of groups of tuples,
particularly across message frames.

Instead, applications should typically consider the last tuple of a
given name as containing the most recent data with respect to a given
message stream.

This looseness allows data to be streamed more efficiently through
Twitter's low-bandwidth messaging platform. Duplicate or non-relevant
values need not be respecified in order to help applications determine
the semantics of collections of tuples.

Note, because of the lack of restriction on tuple delivery order,
applications sending message frames are free to optimize sending of
multiple tuples by arranging them to fit most efficiently across the
fewest number of message frames without truncation.

### Semantic web

We have designed Twitter Data with an eye to layering additional
semantic structures on top. We will discuss these possibilities in a
later version of this proposal.

### Applicability

While its name would seem to indicate a focus only on Twitter, Twitter
Data may also be used on similar messaging services, over instant
messaging (particularly in the case of multicast conferences), email,
SMS, or any other point-to-point, broadcast, or multicast messaging
infrastructure, with or without message size restrictions.

Because of the constraints of Twitter message sizes, Twitter Data is
also intended to be terse and compact without sacrificing human
readability, and to avoid overwhelming normal Twitter streams with ugly,
encoded data intended only for machines. Because of its terseness, it is
useful over low-bandwidth, high-latency, and mobile networks.

Furthermore, because of the human-friendly syntax, Twitter Data is
suitable to be read, injected, and debugged by humans, in the same vein
as HTTP and JSON, making it a convenient, robust, and flexible data
representation format.

Examples
-------------

````
Cruising down 101 near SFO $mph 95 $lat 37.612804 $long -122.381687
````
````
I love the #twitterdata proposal! $vote +1
````
````
Call me $phone 123-456-7890
````
````
Cheap #gas! $lat 37.323144 $long -121.944423 $price 1.99
````
````
#wmodata $id DW1428 $temp 69F $wangle 232 $wspeed 4.0mph $rh 50% 
$dew 49F $press 1015.2mb
````
````
Saw a $aero>man MIG $in walmart$ would you believe it?
````
````
@romeo $foaf>loves @juliet
````
````
$s urn:game123 $move a1b2
````
````
$s urn:roulette123 $bet $$10 $num 5
````

FAQ
---

### Is this really necessary?

Technically, no. However, we do think it has value.

One of the easy criticisms to make about Twitter Data is that equivalent information should be extracted semantically, by analyzing the natural language of user tweets, instead of by users or applications injecting data manually. Ideally, this would be possible, but unfortunately it's not. The technology industry is a long way from truly useful natural language processing, and furthermore, not all types of data can be extracted in that way.

Until we have technology that's at least as smart as a human, structured data that is created, annotated, and curated by humans will always have a place. Will it have a place on Twitter? That's up to you.

### Isn't it a bad idea to litter Twitter feeds with data intended for machines?

Maybe. Probably. Possibly not.

There are a number of possible concerns, but the most common is probably that applications using Twitter Data will spam Twitter with hard-to-read messages and irk all of the human users.

We agree that this may be a short-term possibility. However, in our opinion, the usefulness of embedding data in Twitter messages outstrips any such concerns, and we are willing to live with any short-term discomfort because it ultimately makes Twitter far more useful to its users.

While it may or may not be a defense, assure yourself that Twitter Data must become reasonably popular before these concerns become salient. If Twitter Data does indeed become popular, that will only be a good thing for us all, because it will mean that a new dimension in Twitter has opened up and that there is a broad usefulness in the Twitter ecosytem. At that point, we expect to see Twitter clients that account for this additional channel of data and limit its impact on us poor humans.

### So you're talking about microformats for Twitter?

Actually, no. Don't confuse the role of Twitter Data as a means for embedding abstract data with the uses of Twitter Data to embed particular data.

Microformats are a suite of special-cased conventions for embedding specific types of common data in Web pages. It has limited or no applicability outside of this use case. The conventions it defines are built on top of a lower-level, well-defined formal syntax (HTML) which makes it possible to embed data in various, legal ways.

As a movement, Microformats are far more concerned with trying to standardize representations of common pieces of data than with standardizing how the embedding of that data should be done in the abstract. Thus, it is really a community standardization effort around various ontologies, not a technical standardization effort. The result of this focus is that we see a mishmash of different means for embedding data across the various Microformat specificiations. Microformats is specifically not about a means for an application to define its own "microformat"--a term that makes no sense, since something can't be a microformat without being standardized by community.

Like HTML in the Microformats case, Twitter Data is one abstraction level lower, and concerned with a formal means of making abstract data embeddable in Twitter in a natural and searchable way. What that data actually represents, for example, a microsformat, JSON string, or Base64-encoded bits, is what Twitter Data enables, but not strictly what Twitter Data is about.

The purpose of Twitter Data is to enable community-driven efforts to arrive at conventions for common pieces of data that are embeddable in Twitter by formal means. In other words, "microformats" as community-standardized conventions are completely compatible with Twitter Data and can (and in our opinion, should) be built on top of Twitter Data.

### Why not extend the hashtag syntax instead of defining a new syntax?

Hashtags are specially indexed by Twitter search and are terminated by any non-alphanumeric character. This limits our ability to define a namespace mechanism in Twitter Data.

Furthermore, the use of hashtags is already highly conventional in the Twitter community and it is undesirable to define additional semantics for the use of the existing mechanism. We decided that converting hashtags to tuples instead of unary values was too much of a conflict with current hashtag conventions.

### Why not use JSON for tuple values?

We discussed using JSON, but concluded that it is too verbose for general use, especially with a limit of 140 characters. Also, it doesn't work well with our intent to keep messages human-readable when possible. But, if you really, really want to, you can use JSON in tuple values. Just make sure to escape any embedded dollar signs ("$").

### Why not use XML for tuple values?

XML is way too verbose to fit into 140 characters for really interesting applications. Furthremore, it's very unfriendly to human readers.

Contributors
----------------
### Authors

-   Todd Fast, [@toddfast](http://twitter.com/toddfast),
    todd@toddfast.com,
-   Jiri Kopsa, [@jirikopsa](http://twitter.com/jirikopsa),
    jiri.kopsa@gmail.com

### Reviewers

-   Mike Gionfriddo, [@mgion](http://twitter.com/mgion)
-   Ted Leung, [@twleung](http://twitter.com/twleung)
-   Chun Xia, [@chunxia](http://twitter.com/chunxia)
-   Hao Thai

[![Creative Commons
License](http://i.creativecommons.org/l/by-sa/3.0/88x31.png)](http://creativecommons.org/licenses/by-sa/3.0/)

This work is licensed under a [Creative Commons
License](http://creativecommons.org/licenses/by-sa/3.0/).
