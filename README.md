# Olympian R and Twitter Hackery

*[Noam Ross](https://twitter.com/noamross), 2018-02-09*

I created a [Twitter list of accounts to follow for the Olympics](https://twitter.com/noamross/lists/olympicfollow) and wanted
to bulk-populate it, starting with Twitter's list [of verified Olympic atheletes](https://twitter.com/verified/lists/olympians).
Unfortunately list maintenance via Twitter's web or app UI is a bit of a pain - one has to go
through several clicks for each individual user you want to add to a list, and
there are 103 on Twitter's verified olympian list.

Thankfully, almost all of Twitter's functionality is scriptable via API, and 
R has some Twitter facilities via the [**rtweet**](https://github.com/mkearney/rtweet/)
package.  Unfortunately **rwteet** does not wrap list-modification functions.  I
realized two things, though:

-   Functions not available in **rtweet** but available through the Twitter API can
    be called via other packages such as  [**httr**](https://github.com/r-lib/httr).
-   **rtweet** can still handle authentication and pass a token on to **httr**.

So, here's my bit of morning hackery:
```
# Load libraries. I've skipped over setting up rtweet auth, but you only
# need to do it once, instuctions here:
# https://cran.r-project.org/web/packages/rtweet/vignettes/auth.html

library(rtweet)
library(httr)

# Get a data frame of the lists I follow. I've already created my
# `olympicfollow` list and subscribed to `verified/olympians`
mylists <- rtweet::lists_users("noamross")

# Get a the list ID `verified/olympians`
olympianid <- mylists$list_id[mylists$name == "olympians"]

# Get a data frame of the members of `verified/olympians`
olympians <- lists_members(olympianid)

# Get the list ID of my own list, `olympicfollow`
olfollow_id <- mylists$list_id[mylists$name == "olympicfollow"]

# Build an API call URL. Note the max number of users you can add in 
# one call is 100.
add_list_members_url = paste0(
  'https://api.twitter.com/1.1/lists/members/create_all.json', #Base URL for bulk adding to lists
  '?user_id=',                                                 #paramaeter for users
  paste(olympians$user_id[1:99], collapse=","),             #Comma-separated list of users
  "&list_id=",                                                 #parameter for which list to add to
  olfollow_id                                                  #`olympicfollow` list ID
  )

# Make a POST API call to that URL and use the token that stored by the
# rtweet package to authenticate.
members_response = POST(add_list_members_url, rtweet::get_token())

# Check that you got a successful response
members_response
content(members_response)

# Do it again to get the remainder of the olympians. I suppose if there were
# >200 I would automate this.
add_list_members_url = paste0(
  'https://api.twitter.com/1.1/lists/members/create_all.json', 
  '?user_id=',                                                 
  paste(olympians$user_id[100:103], collapse=","),             
  "&list_id=",                                                 
  olfollow_id                                                  
  )
members_response = POST(add_list_members_url, rtweet::get_token())


```

The list is here!: <https://twitter.com/noamross/lists/olympicfollow>. Who else should I follow? What other lists should I bulk-add?
