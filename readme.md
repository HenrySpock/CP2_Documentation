1. 
The site url is technically 'castlingfe'. 
The app is technically named 'castle tracker.'
The title of the app on the webpage is 'Let's Go Castling!'
The URL is https://castlingfe.onrender.com/

2. 
My website is a lite social media app based on travel, with some stylization towards people who like traveling to medieval sites.

A user 'posts' by logging entries of place they have been as 'travelogs'. 
When a user has multiple related travelogs, they can be grouped as a 'trip.' 
Trips and travelogs can be created as places a user has been or places a user wants to go.
Travelogs can have image urls added to them, trips simply use the first image of the first travelog associated.
Travelogs and trips both allow for blog entries with rich text formatting.
Travelogs and trips can have 'comments' by users, either posted to the trip/travelog or to other comments.

Users can 'like' other users for their profiles, trips, travelogs or comments with different descriptions.
Users can interact directly with others by friending or following, remove other users by blocking, or reporting users to admins.
When users are friends, they can direct message each other.

Admins have an 'admin panel' page specifically for dealing with user reports and scheduling maintenance.
Admins can notify, communicate directly with, suspend, ban, or clear other users of their reports.

Various data from the site (places traveled, likes, number of posts) are collected and shown as leaderboards on the 'abaci' page.
I refer to the site as 'lite social media' because it's not a site where someone can infinitely post about daily errata, posts must revolve
around trips or travelogs.

The site utilizes websocket for direct messaging and notifications, polling for notification/message counting, and cron jobs for several maintenance tasks.

3.
Tech Used
I designed the app the way I did to provide a user experience based on anonymity and defense from abusive posting. People can certainly comment and direct message, but when a user is blocked, neither the blocker or blockee can see anything to do with each other.It's hopefully not the most divisive subject matter. I also wanted to prevent people from posting pictures of their pets or meals or workouts or life tips or weird things they noticed on their walk - people can say anything they want, but it has to be framed in one of the two optional types of post - trip or travelog. Call this my way of trying to 'keep people on topic.'

The app relies on jsonwebtoken for validation and bcrypt for hashing - I saw no reason to reinvent the wheel there.
Some of the date displays rely on 'moment'.

The scheduled tasks use node-cron. The scheduledjobs.js file has 4 functions:

One that sets ignored friend request to 'denied' after a month (which doesn't block the person but prevents them from sending another friend request), delete account suspensions older than 3 days, a check for updating of a maintenance mode length while in maintenance mode, and a function that deletes the previously ended maintance mode but logs it's existince to a 'maintenance history' table before deleting.

Websocket (socket.io) seemed the obvious choice for direct messaging and notifications. It might have been better to use for the polling tasks, but I don't understand it well enough to make it do everything I want yet. The cron jobs for maintenance also seemed the way to go, but again, those are things I have barely scratched the surface with. More traditional polling was used to handle notification / message counting for navbar badges and for tallying site data for the leaderboards (abaci.)

Nodemailer is employed for contacting admins when there is user feedback or when users (or there posts/comments) are reported.

There are several places where the code is clearly not 'DRY', and this was a decision on my part based on which kind of headache I wanted:
did I want the headache of defining a multitude of if/thens to make sure similar tasks were handled correctly, or did I want to copy the code and change the variable names so that I could see where the breakdowns were quicklys? This is also part of why the server.js is so long and not broken out as much as I would have liked - for as big as this project is, I certainly didn't learn 'everything about websocket' or 'everything about react' etc. So, when it came to routes that had notifications, everything was fine when I kept them running from server.js, but when I tried to break out those routes to respective components, things broke down. 

The display of the information on the '/', 'homeother', '/hub', 'public_profile', '/trav_det' and 'trip_det' routes rely on Leaflet maps. I looked into Google and Foursquare and several other map apis but settled on Leaflet because it's a static library. This means no free user accounts that are suddenly getting charged an arm and a leg because navigating an api charge description doesn't make sense. It just works.

I ultimately settled on the Yelp api because of ease/freedom of use and thoroughness of detail (returned). It's only used in one place, but as the primary type of posting on the site is travelogs, it's incorporated in the one place everyone would have to encounter it if they make a post on the site (as trips can't be created without at least 2 travelogs, and travelogs are created with a Yelp search as the first option.) The error for a failed search explicitly tells people to check if Yelp is available in the country searched with a link to the page.

Fontawesome is used for the various like badges.

I tried every rtf formatter I could, and they all failed and I had given up (draft, quill, sunnote, etc.) but then I heard about TipTap and tried it, and was able to get it to work and style. So TipTap is incorproated in the log entries for trips and travelogs - this being where someone writes their actual post beyond just details or images.

4. 
User Flow
/auth
Register - enter details, verify email, then login
Non-logged in users can reset their password by entering email, security question and security answer

/ & /homeother
Upon logging in, users are sent to the home page where they see the publick map of all available trips and travelogs (there are private trips and travelogs which only the user or those granted permission can see.) There are also options to look at the newest user, neweset trip, newest travelog, a random travelog (Surprise Me!) or see 'Other Views' - the main '/' page has sorting based on trip/travelog, visited/want to visit, and categories. The 'Other Views' page has sorting based on whether a site in Unesco, involved in a video game or involved in a film.

/about
A page describing how the site works with a button link to /documentation for a more explicit description.

/abaci
The data leaderboards for the page, with things like Top 5 Liked Travelogs or Top 5 Countries Visited.

/hub
When a user goes to their user hub, they see a similar page to /, but it only includes that users trips or travelogs. From here, a user can edit their profile or see their public profile /public_profile (what other users see of their content.) The user can see who if anyone has liked their profile, and see any notifications about interactions with their content or profile. They can also go to 'see connections' /connections which shows them lists of friends, followers and followees. At the bottom, the user's feed can be sorted based on trips and travelogs by the user, their friends, followers and followees.
From the edit profile page /profile, a user can edit their details, reset their password, turn tooltips on or off, go to /disconnections (to see lists of people they have blocked or friend requests they have denied if they change their mind), or delete their profile.

/log_entry
Going to 'Log an Entry' lets users create a travelog. This starts with a Yelp search. If the site (or country) is not on Yelp, the details can be entered manually. The title, date and image_url are required for sorting and for map markers. Latitude and longitude are required for markers, but if a user doesn't want to look them up, they can use a point and click map here. Once a travelog is submitted, the user is sent to the /trav_det page for that travelog.

/trav_det/travelog_id
This is the page where a user can see a posted log entry with a map showing an icon and location of the site visited - if user is author, they can write a log entry (this is done after the travelog is created becase it kept the creation page much cleaner), edit the images, see who if anyone has liked the travelog, edit travelog details or grant permission for other users to see it if the travelog is private. If user is not author, they can like the travelog or report the travelog.

/create_trip
From here, a user can create a trip, which means 'group of travelogs.' After the trip is created, the user is sent to /trip_det/trip_id. This is similar to trav_det, which the key difference being the map here shows not a single travelog but all travelogs in the trip, numbered oldest to youngest and shown on the map with numbering and polylines. The list of travelogs below is numbered {index + 1}.

/messaging
If a user is friends with someone, they can direct message each other. If a user deletes the conversation, they no longer see it but the other user will. If both users delete the conversation, it's deleted from the database. If a user has an admin as a friend but gets reported, the admin moves out of the friend area to the top and can only communicate as admin until the report is resolved.

Logout does just that.

/admin_panel
Available to admins to deal with user reports and schedule maintenance. Admins can interact reported users by notifying a user (which sends a notification that their account is under review), direct messaging a user, suspending a user which prevents the user from logging in for 3 days, banning a user which deletes all of the user's entered content and adds their email to a banned_emails table, or clearing a report which deletes the report (a user cannot be cleared while suspended.)

5.
I used the Yelp api to help users get details about the sites they visited. I don't think I have anything to add about it.

6. Technology stack:
React / Node / Sequelize / Express / Postgres / Leaflet / TipTap / Render