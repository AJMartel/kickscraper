# Kickscraper

Kickscraper is a library for interfacing with Kickstarter's undocumented/unannounced API. With it, you can get a lot of data about projects, whether you've backed them or not, including reward levels, amount pledged, links to the videos, updates, if it's been successful or not, and details about the creator.

Much of the of data available is documented in the [wiki for this project](https://github.com/markolson/kickscraper/wiki/_pages).


## Status Update

This version now uses Kickstarter's public search for all project searches. It should be a drop-in replacement for the
previous version.


## Installation

    $ gem install kickscraper

Or for use in another app, add it to your Gemfile

    # use the gem
    gem 'kickscraper'
    
    # or stay up to date with the repo (which should be stable)
    gem 'kickscraper', :git => 'git://github.com/markolson/kickscraper.git' 


## Quick way to get started in the console
    
    1. Put your real Kickstarter user credentials in spec/test_constants.rb (or leave blank, which will still allow some kickscraper features)
    2. Enter the console with "rake console"
    3. Start using kickscraper with any of the examples below (starting with "c = Kickscraper.client")


## Sample Usage

Provided with your user credentials this will list the first 20 or so projects you've backed, along with if the project is still active and if it has met it's funding goal.

    require 'kickscraper'

    Kickscraper.configure do |config|
        config.email = 'your-kickstarter-email-address@domain.com'
        config.password = 'This is not my real password. Seriously.'
    end

    client = Kickscraper.client
    puts " A | C |"
    puts "------------------------"
    client.user.backed_projects.each {|x| 
        print (x.active? ? ' X |' : '   |')
        print (x.successful? ? ' X | ' : '   | ')
        puts x.name
    }


## API Examples
    c = Kickscraper.client

    c.user.class
    => Kickscraper::User

    # List what data is available on an object
    c.user.keys
    => ["id", "name", "slug", "avatar", "urls", "location", 
        "biography", "backed_projects_count", "created_projects_count", 
        "unread_messages_count", "unanswered_surveys_count", 
        "starred_projects_count", "social", "send_newsletters", 
        "category_wheel", "notify_of_backings", "notify_of_updates", 
        "notify_of_follower", "notify_of_friend_activity", 
        "notify_of_comments", "notify_mobile_of_backings", 
        "notify_mobile_of_updates", "notify_mobile_of_follower", 
        "notify_mobile_of_friend_activity", "notify_mobile_of_comments", 
        "updated_at", "created_at"]


    # In addition, User types have "backed_projects" and "starred_projects" available.
    c.user.backed_projects
    => [<Project: 'The Veronica Mars Movie Project'>, <Project: 'RiffTrax Wants to Riff TWILIGHT Live in Theaters Nationwide!'>, <Project: 'To Be Or Not To Be: That Is The Adventure'>, <Project: 'Planetary Annihilation - A Next Generation RTS'>, <Project: 'OUYA: A New Kind of Video Game Console'>, <Project: 'Gotham Knight Terrors: Comedic Batman Short'>, <Project: 'Internet Meme Playing Cards'>, <Project: 'GOOD JOB, BRAIN! - A Trivia & Quiz Show Podcast'>, <Project: 'Elevation Dock: The Best Dock For iPhone'>, <Project: 'Trebuchette - the snap-together, desktop trebuchet'>]

    # Some values are already converted to their appropriate types. Some aren't yet.
    c.user.backed_projects.first.creator.class
    => Kickscraper::User

    # You can chain methods together and dig into the objects.
    # So get the thumbnail of the person that created the latest project we backed
    c.user.backed_projects.first.creator.avatar.thumb

    # You can search for projects
    c.search_projects('veronica mars')
    => [<Project: 'The Veronica Mars Movie Project'>, <Project: 'Prodigal Daughter - TV Show Pilot'>]

    # and view their rewards or updates, in addition to any of the data found in the "keys"
    vm = c.search_projects('veronica mars').first
    vm.updates
    => [...]
    vm.successful?
    => true
    vm.video.high
    => "https://d2pq0u4uni88oo.cloudfront.net/projects/56284/video-217182-h264_high.mp4?2013"

    # When searching for projects (or using convenient methods like recently_launched_projects()),
    # you can continue the search until Kickstarter returns no more results
    c.recently_launched_projects
    => [ _array of projects_ ]
    c.load_more_projects if c.more_projects_available?
    => [ _array of additional projects_ ]

    # Print all the updates for all the current user's projects
    c.user.backed_projects.each { |project|
        puts project.name.upcase
        project.updates.reverse.each { |update|
            # strip the HTML out of the body, since this outputs to a terminal
            puts "Update #{update.sequence}: #{update.body.gsub(/<\/?[^>]*>/, "")}\n\n"
        }
        puts "\n\n"
    }
    
    # Get info about Kickstarter categories or find projects by category
    c.categories
    => [<Category: 'Art'>, <Category: 'Conceptual Art'>, <Category: 'Crafts'>, <Category: 'Animation'>, <Category: 'Tabletop Games'>, <Category: 'Classical Music'>, <Category: 'Art Book'>, <Category: 'Hardware'>, <Category: 'Comics'>, <Category: 'Digital Art'>, <Category: 'Graphic Design'>, <Category: 'Documentary'>, <Category: 'Video Games'>, <Category: 'Country & Folk'>, <Category: 'Children's Book'>, <Category: 'Open Software'>, <Category: 'Dance'>, <Category: 'Illustration'>, <Category: 'Product Design'>, <Category: 'Narrative Film'>, <Category: 'Electronic Music'>, <Category: 'Fiction'>, <Category: 'Design'>, <Category: 'Journalism'>, <Category: 'Painting'>, <Category: 'Short Film'>, <Category: 'Hip-Hop'>, <Category: 'Fashion'>, <Category: 'Performance Art'>, <Category: 'Webseries'>, <Category: 'Indie Rock'>, <Category: 'Nonfiction'>, <Category: 'Film & Video'>, <Category: 'Jazz'>, <Category: 'Periodical'>, <Category: 'Food'>, <Category: 'Pop'>, <Category: 'Poetry'>, <Category: 'Mixed Media'>, <Category: 'Games'>, <Category: 'Rock'>, <Category: 'Public Art'>, <Category: 'Radio & Podcast'>, <Category: 'Music'>, <Category: 'Sculpture'>, <Category: 'World Music'>, <Category: 'Photography'>, <Category: 'Publishing'>, <Category: 'Technology'>, <Category: 'Theater'>] 

    category = c.category('product design')
    => <Category: 'Product Design'> 

    category.keys
    => ["id", "name", "position", "parent_id", "parent", "projects_count", "urls"] 

    category.projects_count
    => 234 

    ps = category.projects
    => [<Project: 'The Skinny Mirror - The Mirror That Compliments You'>, <Project: 'Rustic Fort'>, <Project: 'pushXpro: Revolutionizing the push-up exercise'>, <Project: 'Introducing Nubrella, the world's first hands-free umbrella.'>, <Project: 'Stream: The Pebble Smartwatch Dock'>, <Project: 'Get iPatched - Protect Yourself From Big Brother & Hackers'>, <Project: 'TrueRec introduces the DFP Kayak - Dive Fish Paddle'>, <Project: 'MagCozy: A Leash for Your MagSafe 2 Adapter'>, <Project: 'ECO Friendly Bamboo Party Trays'>, <Project: 'PJ reefs Miniature Saltwater Aquarium'>] 


## Contributing

There are two good ways to contribute: 

First, by using it and creating Issues as you find problems or rough spots. Second, by taking matters into your own hands:

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
