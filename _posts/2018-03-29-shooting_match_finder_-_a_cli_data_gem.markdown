---
layout: post
title:      "Shooting Match Finder - A CLI Data Gem"
date:       2018-03-29 13:39:20 -0400
permalink:  shooting_match_finder_-_a_cli_data_gem
---


Alright! Today we're going to walk through the creation of Shooting Match Finder, my CLI RubyGem that displays information about the top shooting competitions around the country! I'll go over some of the choices I made that led to the eventual code I've written. Let's get started!

**The Initialization**

From the beginning, I knew I wanted to scrape [Practiscore](http://practiscore.com/search/matches) for my match information. Practiscore is one of the most widely used match scoring databases available for competitive shooters (I've got scores on there myself!), and the match page structure lends itself very well to scraping. With the website I wanted to use decided, the time for code had come.

I chose to work "backwards" (at least that's how I felt about it) and start from the CLI interface, with what users of my program would see. I figured I could use hard coded data, and just move it into other methods and files, gradually growing my program from the interface side so that once I finished the Scraper file and started plugging in real data, the rest of the program would just work. I used Bundler to create the basic framework and environment for my Gem, which was a huge lifesaver. With that out of the way, I could start digging into my programming!

First, I needed a #start method, which called all the other methods contained in my CLI. Avi Flombaum always says, "Write the code you wish you had." Taking this to heart, I started by building my CLI the way I thought it should work, and then went from there.

> Write the code you wish you had.

**The CLI**

Overall, the CLI interface code is pretty simple. We just instansiate match objects using some methods to call the Match and Scraper class (which we'll get to a little further on). The bulk of the work is done by #menu, which you can see below.

```
def menu #Is able to show details about any match.
    input = nil
    while input != "exit"
      puts "Enter a match number for more info, list to see matches, or type exit."
      input = gets.strip
      if input.to_i > 0 && input.to_i < Match.show_matches.length
        puts ""
        puts "#{Match.show_matches[input.to_i - 1].name}"
        puts "  Start Time: #{Match.show_matches[input.to_i - 1].match_start}"
        puts "  Location:   #{Match.show_matches[input.to_i - 1].location}"
        puts "  Entry Fee:  #{Match.show_matches[input.to_i - 1].entry_fee}"
        puts "  #{Match.show_matches[input.to_i - 1].description}"
        puts ""
      elsif input.downcase == "list"
        list_matches
      else
        puts "Please enter a match number, or type exit."
      end
    end
  end
```
While it looks like a lot, digging into it, it simply takes a user's input, and if that input is a number that corresponds to an index in the array of the Match class' @@all class variable, it will puts out the information about that match to the terminal. The menu is also able to discern valid input from invalid, as well as list the matches again if the user ever needs them. Typing "exit" ends the program. Moving on!

**The Match Class**

With the CLI well in hand and responding well, the next thing we need is a class capable of creating objects to pass to the CLI. Enter the Match class.

```
class Match
  attr_accessor :name, :location, :entry_fee, :match_start, :description, :match_url

  @@all = []


  def initialize(match_hash) #takes in a hash to assign each match a name and a url for later use in #add_attributes.
    match_hash.each {|key, value| self.send(("#{key}="), value)}
    @@all << self
  end

  def self.new_from_practiscore(match_array) #Uses the hash array passed in by cli.rb to create matches.
    match_array.each{|match_hash| Match.new(match_hash)}
  end

  def add_attributes(match_info) #takes in a hash and uses mass assignment to add match details.
    match_info.each {|key, value| self.send(("#{key}="), value)}
  end

  def self.show_matches
    @@all
  end
end
```
That's it! The whole thing! This is the final version. Initially, I filled these methods with "fake data", hard-coded information in the format the CLI needs, just to make sure my classes were communicating correctly. As the program grew, I slowly phased out that fake data, or moved it into the Scraper class (which we'll go over next), as a way to test my program without using a full rspec test suite.  

The Match class as a whole is pretty self-explanatory. Each match object is initialized with a hash as an argument. The hash contains all of the information created by the Scraper class, and uses mass assignment logic to give attributes to each specific match object. You can see that code repeated in #add_attributes, which is an instance method that gets passed a secondary hash from Scraper to create the match details (which are stored in another webpage on Practiscore. More on that later.). Also worth noting is #new_from_practiscore, which takes in an array of hashes as it's argument to batch create Match objects, thus automating the whole process. Other than that, we just need a way to access @@all outside of the Match class, hence the #show_matches class method. On to the meat of the project!

**The Scraper Class**

Oh baby, this is where the magic happens. Let's go through it! 

First things first, we have the initial scraping method, #scrape_matches. This method is responsible for grabbing the name and date of the match, as well as its unique href attribute, which is passed to the second Scraper method, #scrape_from_match_url, which we'll talk about soon.

```
def self.scrape_matches(website_url) #This creates an array of hashes that we then use to create match objects.
    doc = Nokogiri::HTML(open(website_url))
    matches = doc.css(".list-group-item")
      matches[0...51].collect do |match_details|
        {:name => match_details.css(".searchMatchWebName").text.gsub("Open", "").gsub("Closed", "").strip,
        :match_url => match_details.css("a").attr('href').text}
    end
  end
```

Notice how the matches array specifies ONLY array indexes 0-51. That's because, when I first ran the program, I got over 1,200 match objects! Unbeknownst to me, the way the Practiscore website is formatted only shows you 50 matches at a time, unless you click the small "Show more" button at the bottom of the list, but every single match is still stored in the website HTML! So I chose to only show the first 51, as that's what the base page of Practiscore looks like.
	
Another, supremely frustrating pitfall I ran into was the fact that the "Show more" button I mentioned before *also* has an HTML class of .list-group-item, meaning it was being included in my scrape! It was breaking my program because it didn't have an href attribute, which was giving me really strange Nokogiri errors. That error took me seriously three days to figure out, and I felt simultaneously so relieved and so stupid once I finally figured out what was happening. However, I now have valuable experience in scraping from a table, so I consider it a victory!
	
With this method working correctly, we move on to the second Scraper method, #scrape_from_match_url.

```
def self.scrape_from_match_url(match_url) #Used to add attributes to newly created match objects.
    doc = Nokogiri::HTML(open(match_url))
    #This logic is used to identify matches based on whether or not registration is open, since they need to scrape differently.
    if !doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[1]').text.include?("Registration opens")
      match_info = {:match_start => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[1]/strong').text.strip,
                    :location => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[2]').text.gsub("Location:", "").strip,
                    :entry_fee => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/dl/dd/text()').text.strip,
                    :description => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[3]').text}
    else
      match_info = {:match_start => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[2]/strong').text.strip,
                    :location => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[3]').text.gsub("Location:", "").strip,
                    :entry_fee => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/dl/dd/text()').text.strip,
                    :description => doc.xpath('//*[@id="psMainContainer"]/div[2]/div[3]/div[1]/p[4]').text}
    end
  end
```

Wow, that looks a lot more impressive. This method made my wife shake her head and ask in disbelief if I actually knew what any of that meant. Let me tell you, being able to tell her *exactly* what this method does was so, so satisfying.

Let's talk about it!

This Scraper class method takes in the match_url given to it by a match object, and then scrapes from that particular match's detail page. I specify a BASE_PATH constant in my CLI of "https://practiscore.com" and append on the specific match url when this method is called, making sure I always get the right match page and saving me some typing.

Other than that, it's a pretty cut and dry scraping method, storing information as a hash, which, if you remember, we later use in the Match class #add_attributes method. The unique thing about this method is that I had to scrape twice.

***TWICE***

Which was annoying. Turns out, if a match hasn't yet opened it's registration, there's additional information displayed at the very top of the details page! Since my xpath descriptors are based on specific div and p tags, a match that has that extra info needs to be scraped differently! Otherwise, I was getting all sorts of strange behavior from my program. Certain matches would display exactly as I wanted them to, others would display wrong information. Luckily, Practiscore has standardized the way it shows this additional registration information, so I was able to employ some simple logic to determine whether or not a match had opened registration. But I still had to do scraping code twice.

**IT LIVES!**

With everything finished, all I had to do was execute the program and voila! A list of neatly formatted shooting competitions, with the ability to view each match's details at the push of a button. The feeling of pride and accomplishment that welled up inside me the first time I successfully ran my Gem was intoxicating. Knowing that I had built this completely unique, fully functioning program from a blank file was just amazing.

I learned so much during this process as well! Setting up environments with Bundler, xpath, building a robust CLI interface... The journey taught me a ton of practical knowledge.

I can't wait to build another!

![](https://media.giphy.com/media/ziLadIVnOGCKk/giphy.gif)

