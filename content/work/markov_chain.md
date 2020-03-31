---
title: "Using 1.6 Million Tweets with a Markov Chain Text Generator"
date: 2019-02-19T18:18:25-05:00
tags: ["R", "projects"]
---

During my daily dose of reddit browsing, I came across [this post](https://www.reddit.com/r/SubredditSimulator/comments/91pjrh/the_view_from_my_offices_toilet_today/)
and was subsequently pretty confused. *How can an office be THAT high?* Amidst
my bewilderment, I noticed that it came from the subreddit  
[SubredditSimulator](https://www.reddit.com/r/SubredditSimulator/), and its
hilarity immediately grabbed my attention. If you are unfamiliar with how Reddit
operates, it is basically a giant forum split into thousands of subcommunities
called "subreddits." Users are allowed to subscribe to whatever subreddits they
want to so they can regularly view its content, create posts in any subreddit,
and comment on any post they like.

But /r/SubredditSimulator is a little different. Regular users can't make their
own posts or comments, however they still play a role in the voting system for
content. The users that generate the actual content  are all bots who use
[Markov Chain text generation](https://en.wikipedia.org/wiki/Markov_chain#Markov_text_generators)
to create their content, hence the seemingly nonsensical titles and comments.
The bots are all modeled after a certain subreddit, and it's text data comes
solely from their home. For example, /u/Jokes_SS is meant to mimic posts and
comments found on /r/jokes.

## But what is a Markov Chain?

Basically, a Markov Chain is a way to model a stochastic process of one state
transitioning to its next state. It is important to note that future outcomes
only depend on the current state of the model, and not past events (this is the
memoryless property). Wikipedia uses this image to  illustrate this idea:

![https://en.wikipedia.org/wiki/Examples_of_Markov_chains ](/images/markov_chain.png)

The two nodes on this graph (sunny or rainy) represent two states that the model
can be in, while the edges and their respective numbers represent the probabilty
of the transition from one state to the next (in this case the probability of
the forecast on the following day). Let's say today is Sunny. There is a 90%
chance it will be sunny on the tomorrow, while there is a 10% chance  tomorrow
will be rainy. It doesn't matter if yesterday was sunny or rainy, because today
it's sunny, and that's all that matters when considering tomorrow's weather.


This same process can be applied to text generation processes. Let's make some
data!

    1. "I pet the dog."<br>
    2. "I have a pet dog."<br>
    3. "I went to the pet store."<br>
    4. "I pet the pet cat."<br>

Great! now let's build a sentence. We will start with the letter "I" for
simplicity purposes.

    I

If we fed this into a Markov Chain function, it would build a data structure
that has a key word (in this case it is "I"), and it's values would be all of
the words that followed the word "I" in our given data.

    1. "I <mark>pet</mark> the dog."<br>
    2. "I <mark>have</mark> a pet dog."<br>
    3. "I <mark>went</mark> to the pet store."<br>
    4. "I <mark>pet</mark> the pet cat."<br>

Once all of the words are found, the function calculates a proportion of how
often the second word follows the first word. Using the weather example still,
the current word we are using in our sentence would be equivalent to today's
weather, and the next word would be tomorrow's weather. So since we started with
"I", the words that follow it in our data are "pet", "have", "went", and "pet"
again. The data might have this sort of map-esque structure (note that this is
just the "I" data):

    {"I": {"pet":0.5, "went":0.25, "have":0.25}, ...}

So we have the word we are currently using at the front, and then each
successive word and its probabilities. The function would then randomly choose a
word using these probabilities, and then add it to the sentence. Let's say it
chose "pet", so it's added after "I."

    I pet

It  then creates the transition map for "pet":

    {"pet":{"the":0.4, "dog.":0.2, "store.":0.2, "cat.":0.2}}

and randomly selects a work and adds to our sentence:

    I pet store.

It doesn't really make sense, but that's why it's so fun. Now that we understand
the basics of Markov Chains, let's open up R and implement it there.


## R Implementation

We will make use of the [ngram R package](https://cran.r-project.org/web/packages/ngram/ngram.pdf)
because it has a Markov Chain function named babble() that we will make use of.
Our data comes from [this Kaggle dataset](https://www.kaggle.com/kazanova/sentiment140),
which contains 1.6 million tweets, which is great for large amounts of word pair
associations to create, however there is a trade-off in how long it will take
for our script to run. Small datasets are not ideal for generators simply
because there will not be as many word pairs to randomly choose from, and
sentences will look very similar to the original ones fed into the function.
Here is a script I wrote to return a tweet using this data.


    library(tidyverse)
    library(ngram)
    library(stringi)

    tweets <- read_csv("training.1600000.processed.noemoticon.csv", col_names =
        c("target", "ids", "date", "flag", "user", "text"))

    # Encode weird byte issues so it concatenate actually runs
    tweets$text <- stri_enc_toutf8(tweets$text, is_unknown_8bit = TRUE, validate = TRUE)

    # Eliminate tweets longer than the max character limit
    tweets <- tweets %>% filter(nchar(text) <= 280)
    # Turn the column containing all of the tweets into one
    # string to be fed into the babble function
    # This takes a while to run due to the size of the data.
    # This took my 2015 Macbook Pro  51.25 seconds to run, but your mileage may vary
    str <- concatenate(tweets$text)

    # Create the ngram obejct to be passed into the babble function
    ng <- ngram(str)

    # This function takes an ngram object and an integer that specifies number of words to return.
    # The trim argument will trim off excessive words after the
    # last punctuation point, and makes the tweet a little more understandable.

    # Input: ngram object, integer, boolean
    # Output: String

    make_tweet <- function(ngram = ng, num = 35, trim = TRUE){
      stopifnot(num <= 35)

      # Generate Tweet
      str <- babble(ngram, genlen = num)

      # remove mentions, hastags, links, and fix quote and ampersand encoding issue
      search_patterns <- c("@\\w+ *", "#\\w+ *", "https\\w+ *",
		       "http\\w+ *", "Http\\w+ *", "HTTP\\w+ *",
                           "www.\\w+ *","://t.co/\\w+)", "://bit.ly/\\w+ *",
                           "&quot;", "&amp")

      replace_patterns <- c(rep("", length(search_patterns) - 1), "and")

    # This website helped me with mapply
    # https://bit.ly/2T7Rz5Q

      mapply(function(u, v) str<<-gsub(u, v, str), search_patterns, replace_patterns)

      # Cut off sentence fragments at end.
      if (trim == TRUE) {str <- gsub(x = str, pattern = "[^.]+$", "")}

      # Capitalize first letter
      substr(str, 0, 1) <- str_to_upper(substr(str, 0, 1))

      # Remake tweet if too long, too short, or empty
      if (nchar(str) > 280 || nchar(str) <= 5 || nchar(str) == 0) {
        # Reset str so it doesn't print
        str = ""
        make_tweet(ngram, num)
      }

      # Print it!
      if (str != "") {
        print(noquote(str))
      }
    }

    make_tweet()


Let's see some tweets that it generated!

    Know!! I wish I were him I'd rather stay with them but then again, i'm dreading thursday why doesn't that count? i would like to hear any insults! My guitar is in LA

    So. Thankfully Disney channel offers wonderful programming for multi-processor systems! I think I'm immune to the park with cath...what a beautiful day outside watchin some unfabulous episodes... i couldnt sleep last night too.

    Iz was fussing all night xxx i'm just so i told my exam now. crap. i forgot about bones even though it's night at ours... kids, popcorn, cosy slippers, life is short...

## It's as simple as that!</h2>

Everything looks like it is working properly, which is always nice. I hope you
enjoyed this post, feel free to contact me via Twitter or email if you have any
questions or feedback. I would love to see other ideas or fun data sets to use
with this code.

*Disclaimer: The content and ideas expressed in the tweets are not my own.
