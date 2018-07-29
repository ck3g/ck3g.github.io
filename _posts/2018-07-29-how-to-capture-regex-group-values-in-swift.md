---
layout: post
title: "How to capture Regex group values in Swift"
desc: "Learn how to match and capture regular expression groups values in Swift programming language"
keywords: "Swift, Regular Expressions, Regex"
tags: [Swift, Regex]
excerpt_separator: <!--more-->
---

Today, I need to parse a string of a specific format and grab a couple of values from the string using Swift programming language.
Sounds like a trivial problem. All I need to do is to use a regular expression to find the information I need.

That post would not have written unless I had encountered a couple of difficulties.

I have to admit that I am relatively new to Swift and some parts of the language are not that obvious to me and probably for some beginners.

Yet another issue I've faced today is the Swift documentation.
The documentation looks incomplete.
Although, I might have a bad luck using documentation related to regular expressions.

Anyway. Let's jump ahead and see how I've solved that issue.
I hope that solution can help someone as well.

<!--more-->

Imagine we have a list of video series.
Those series contain a title, a season number, and an episode number.

We are fetching that information from some sort of remote API.
Once we fetch it, we need to parse it and populate the `FilmSeries` structure.

```swift
struct FilmSeries {
  var title: String
  var season: Int?
  var episode: Int?
}
```

The problem is, that API can provide a different structure for different shows.

In one case we can have a title, a season number and an episode number as a separate field.
In another case we can only have a title with the following format: "Season N Episode M - A title of an episode".

So we want to normalize that data and store it properly in the `FilmSeries` structure.

Using regular expressions to parse the title seems like a proper way. All we need is to capture the `N` and `M` values.

Let's get started.

First, we need an example title we are going work with:

```swift
let title = "Season 1 Episode 3 - When Joey meets Zoey"
```

From that title we need to extract values "1" and "3" as a season and an episode numbers respectively.
The regular expression pattern would look like:

```swift
let pattern = "^Season\\s+(\\d+)\\s+Episode\\s+(\\d+)"
```

Let's try to figure out what does that pattern do:

* First we are looking for the text "Season" at the beginning of the string. The `^` indicates that.
* It should be followed by at least one space character (`\\s+`).
* Then followed by at least one number `\\d+`. We are also wrapping that value into parenthesis because we are going to capture it `(\\d+)`.
* Then the expression followed by space characters again
* Then a hard-coded "Episode" string followed by space characters and the last group.

As a next step we would need to create a regular expression object. In Swift, we need to use the `NSRegularExpression` class.

```swift
let regex = try? NSRegularExpression(
  pattern: pattern,
  options: .caseInsensitive
)
```

Now we have the object which uses the specified pattern.
`.caseInsensitive` is short for `NSRegularExpression.Options.caseInsensitive` and indicates that our regular expression object should ignore case difference.

The `NSRegularExpression` class provides several methods to find matching expressions.
In our case, that match is not repetitive.
That means if there is a match we would have only one.

That's why we are going to use `firstMatch` method:

```swift
if let match = regex?.firstMatch(in: title, options: [], range: NSRange(location: 0, length: title.utf16.count)) {

}
```

Here we are looking for a match in the `title` variable, with no additional options, within the range from the beginning of our string till the very end.

The `match` object now which contains additional information.
Using that pattern against our string we should have at least 3 pieces of information:

1. We have the whole match. It is a piece of the string satisfying the regular expression pattern. "Season 1 Episode 3" in our case.
2. We have the value of the first group: "1"
3. We have the value of the second group: "3"

We can access the value of every group by its index. Let's experiment a little bit.

To get the whole matched value, we can use the following construction:

```swift
if let wholeRange = Range(match.range(at: 0), in: title) {
  let wholeMatch = title[wholeRange] // => "Season 1 Episode 3"
}
```

second:

```swift
if let seasonRange = Range(match.range(at: 1), in: title) {
  let seasonNumber = title[seasonRange] // => "1"
}
```

If you haven't noticed it contains the different index in the `match.range(at: 1)` expression.
As you can figure out, to get the episode number we can write:

```swift
if let episodeRange = Range(match.range(at: 2), in: title) {
  let episodeNumber = title[episodeRange] // => "3"
}
```

So we are kinda done. That is how we can fetch the values we need.

```swift
let title = "Season 1 Episode 3 - When Joey meets Zoey"
let pattern = "^Season\\s+(\\d+)\\s+Episode\\s+(\\d+)"
let regex = try? NSRegularExpression(pattern: pattern, options: .caseInsensitive)

if let match = regex?.firstMatch(in: title, options: [], range: NSRange(location: 0, length: title.utf16.count)) {
  if let seasonRange = Range(match.range(at: 1), in: title) {
    let seasonNumber = title[seasonRange]
  }

  if let episodeRange = Range(match.range(at: 2), in: title) {
    let episodeNumber = title[episodeRange]
  }
}
```

Before you go. Let's try to refactor the code a little bit.

First, accessing the captured groups by its index does not seem to be very readable.
We can provide a name to every group in the regular expression pattern.

Let's change it to be:

```swift
let pattern = "^Season\\s+(?<season>\\d+)\\s+Episode\\s+(?<episode>\\d+)"
```

`?<season>` inside parenthesis stands for a group name

And now we can update our code to use those names:

```swift
if let seasonRange = Range(match.range(withName: "season"), in: title) {
  let seasonNumber = title[seasonRange] // => "1"
}

if let episodeRange = Range(match.range(withName: "episode"), in: title) {
  let episodeNumber = title[episodeRange] // => "3"
}
```

Next, if we are going to use that code more than one time it makes sense to extract it into a separate method and then call it every time we need.

```swift
func findSeasonAndEpisodeFrom(title: String) -> (Int?, Int?) {
  var seasonNumber: Int?
  var episodeNumber: Int?

  let pattern = "^Season\\s+(?<season>\\d+)\\s+Episode\\s+(?<episode>\\d+)"
  let regex = try? NSRegularExpression(pattern: pattern, options: .caseInsensitive)

  if let match = regex?.firstMatch(in: title, options: [], range: NSRange(location: 0, length: title.utf16.count)) {
    if let seasonRange = Range(match.range(withName: "season"), in: title) {
      seasonNumber = Int(title[seasonRange])
    }

    if let episodeRange = Range(match.range(withName: "episode"), in: title) {
      episodeNumber = Int(title[episodeRange])
    }
  }

  return (seasonNumber, episodeNumber)
}
```

That method returns a Tuple with required values.

Finally, we can use it in our code:

```swift
let (season, episode) = findSeasonAndEpisodeFrom(title: "Season 1 Episode 3 - When Joey meets Zoey")
let series = FilmSeries(title: series.title, season: season, episode: episode)
```

You can find the complete example in [the following gist](https://gist.github.com/ck3g/d735c0ea50c228144a18cd2915e9ddd0).

## Wrapping up

The lack of documentation and some not obvious parts does not make Swift a bad language.
I believe the understanding would come with experience. By solving that kind of problems, we would become more experienced.
At the moment I hope that article will be able to help someone.
