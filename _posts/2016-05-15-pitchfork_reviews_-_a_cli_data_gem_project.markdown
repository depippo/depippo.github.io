---
layout: post
title:  Pitchfork Reviews - A CLI Data Gem Project
date:   2016-05-15 00:57:13 +0000
---

At the end of the Object Oriented Ruby unit on Learn.co, we are given the task of creating an original RubyGem with a Command Line Interface. More specifically, the gem must scrape the web for data. Once presented with this challenge, I started thinking about all the websites I frequent. Before long, I came to my most trusted resource for music reviews, [Pitchfork](http://www.pitchfork.com) Music is a huge part of my life, and I love discovering new albums. Perhaps I could build a gem that would pull all the latest album reviews from Pitchfork into a simple CLI? Sounded fun and doable enough, so that's what I set out to do.

Becasue this would be the first gem I had built from scratch, I started by watching a very helpful [Railscast](http://railscasts.com/episodes/245-new-gem-with-bundler?autoplay=true) about creating a new gem with [Bundler](http://bundler.io/). I liked the Bundler approach a lot, because within just a few minutes, I was all set up with the basic file structure I would need to build and publish a working gem.

My first step from there was to build a basic functioning CLI. Before getting into the nitty gritty of scraping Pitchfork for data, I at least had a rough idea of how the interface would work, and I could get that up and running quickly. I knew I wanted the program to start by loading a list of the most recent albums reviews, and give the user the option to select any one of them to see more information about it. Pretty simple, and before I had all the behind the scenes logic working, I had something like this in place (with the album variables just hard-coded placeholders at this point):

```
class PitchforkReviews::CLI

  def call
    list_reviews
    menu
    goodbye
  end

  def list_reviews
    puts
    puts "Welcome! Here are the most recent album reviews from Pitchfork."
    puts
    @albums = PitchforkReviews::Album.scrape_pitchfork
    @albums.each.with_index(1) do |album, i|
      puts "#{i}. #{album.name} by #{album.artist}#{album.best_new_album}"
    end
  end
  etc . . .
```
The next step was to think about what other attributes the album objects would need to have. Name and artist were obvious. Additonally, Pitchfork tags certain particularly worthy albums with the Best New Album badge, so I immediately knew that was something I wanted to incoporate. I figured those 3 attributes would be sufficient for the initial list of albums the user sees upon startup, like so:

 1. The Colour in Anything by James Blake - Best New Album - 
 2. Cult Following by Little Scream
 3. Kindness for Weakness by Homeboy Sandman
 4. For Good by Fog
 5. WIll by Julianna Barwick

etc . . .

But if a user wanted further information about each of those albums, what should that look like? After spending some time looking through a few review pages from the site, I settled on genre, score, summary, and url, so that by selecting an album, the user could see something like:

The Colour in Anything
by
James Blake
Pop/R&B, Electronic

Score: 8.2  - Best New Album -

Summary:
Clocking in at 76 minutes, The Colour in Anything is James Blake's wonderfully messy dive into maximalism.

Read the full review at http://pitchfork.com/reviews/albums/21906-the-colour-in-anything/

Now armed with a basic functioning CLI and a list of the attributes I wanted my album class to have, I set out scraping Pitchfork for data. Fortunately, the site is built in a pretty logical manner, such that each individual review is contained in a div with class="review". Knowing this, I was able to start building out the scraper.

```
def self.scrape_pitchfork
    albums = []
    doc = Nokogiri::HTML(open("http://pitchfork.com/reviews/albums/"))
    doc.css(".review").each do |review|
      album = self.new
      album.name = review.css(".title").text
      album.artist = 
      album.genre = 
      album.score = 
      album.summary = 
      album.best_new_album =
      album.url =
      
      etc . . .
```
Certain attributes proved trickier to scrape than others (artist and genre, for example, necessitated iterators of their own for cases where an album had multiples entries), but it was all doable. The score and summary information is only available on each album's specific review page, so to retrieve those, I had to scrape a level deeper, using the album.url for each album:

```
def self.scrape_pitchfork

  . . .

  album.url = review.css(".album-link")[0]["href"]

  page = Nokogiri::HTML(open("http://pitchfork.com#{album.url}"))
  album.score = page.css(".score-circle").text
  album.summary = page.css("div.abstract").text

  etc . . .
```
Once I had my scaper method working, properly retrieving all of my desired album attributes, it was time to return to the CLI and flesh it out. The list_reviews method I already had was still fine, but now I could build out the menu method further. Having learned that the Pitchfork reviews page contains 24 reviews at a time, I was able to make the desired user input more specific.

```
def menu
    input = nil
    while input != "exit"
      puts "Please enter the number of the album you would like to read more about."
      input = gets.strip.downcase
      if input.to_i.between?(1,24)
        the_album = @albums[input.to_i-1]
        puts "#{the_album.name}"
        puts "by"
        puts "#{the_album.artist}"
        puts "#{the_album.genre}"
    
        etc . . .
```
Finally, once I had all of this working, I thought it would be fun to add an option for the user to sort the list of albums by review score. Since each album now had a score attribute, I was able to build a sort_by_score method to be called if the user input 'sort':

```
def sort_by_score
  @albums = PitchforkReviews::Album.scrape_pitchfork
  @albums.sort! { |a,b| b.score <=> a.score}
  @albums.each.with_index(1) do |album, i|
   puts "#{i}. #{album.name} by #{album.artist}#{album.best_new_album}, #{album.score}"
  end
 end
```
Which would return:

1. A Moon Shaped Pool by Radiohead - Best New Album -, 9.1
2. Heartbreaker (Deluxe Reissue) by Ryan Adams, 9.0
3. Will by Julianna Barwick, 8.2
4. The Colour in Anything by James Blake - Best New Album -, 8.2
5. Bottomless Pit by Death Grips, 8.1

etc . . .

After some testing to make sure everything was working the way it should, I pushed my first gem up to RubyGems! I had a lot of fun with this project. Although I'm certain there are ways I could improve my code, it's a really tremendous feeling to have built something of my own, that works the way I intended.

The full code is on github https://github.com/depippo/pitchfork-reviews and available from RubyGems via `gem install pitchfork_reviews`.
