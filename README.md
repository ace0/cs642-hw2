# CS 642: Computer Security - Homework Two

This homework assignment covers exploitation of web security. You must work with a partner. There are four parts; all are required.

It is due **March 28, 2016** by 11:59 PM local time. 

## Web Security Project
You are given the source code for a banking web application written in Ruby on Rails and you will implement a variety of attacks.
The web application lets users manage bitbars, a new form of crypto currency. Each user is given 200 bitbars when they register for the site. They can transfer bitbars to other users using an intuitive web interface, as well as create and view user profiles. You have been given the source code for the bitbar application. Real web attackers may or may not have access to the source code of the target website, but having source might make finding the vulnerabilities a bit easier. Note that we will grade assignments on an unmodified copy of the target web application.

## Setup
A VirtualBox image with Ubuntu Desktop (14.04 LTS), Ruby on Rails, and the bitbar application is available at:
- OVA: http://pages.cs.wisc.edu/~agarwalt/Assignment2.ova

Note this file is YUUGE! The OVA is almost 2GB. Downloading it from a campus internet connection may be much faster than downloading it from a home internet connection. I found (on a Mac) that I could create a new virtual machine marked Linux/Ubuntu 64-bit, add the VMDK file, and skip downloading the OVA file and it worked well. This hasn't been tested on all platforms.

If you like, you can also setup your own Ruby on Rails installation (ruby 2.0, rails 4.0) and clone this repo to get the bitbar application.

As with the previous assignment, this Ubuntu image has a user account (`user`) with password `user`.

The bitbar source code can be found in `~user/cs642/bitbar`. To start the web application, open a terminal and run the commands:
```
cd cs642/bitbar
rails server
```
Then open a web browser and navigate to the URL `http://localhost:3000`. You can close the server by pressing `Ctrl+C` in the terminal. If you do make changes to the server code you do NOT need to restart the server every time you make a change to the Rails source; the running Rails server will automatically update the website when you make a change to the source code.

If you see the following error message:
```
A server is already running. Check /home/user/cs642/bitbar/tmp/pids/server.pid
```
From the bitbar directory run the following commands to delete a stale pid file and restart the rails sever:
```
rm tmp/pids/server.pid
rails server
```

## Attack A: Cookie Theft
Your solution is a URL starting with `http://localhost:3000/profile?username=`
The grader will already be logged in to Bitbar before loading your URL. Your goal is to steal the user’s bitbar session cookie and send it to `http://localhost:3000/steal_cookie?cookie=...cookiedatahere...`. You can view the most recently stolen cookie using this page: `
```
http://localhost:3000/view_stolen_cookie
```
Only the first 64 characters of your stolen cookie will be stored/displayed. This is expected and ok.

The attack should not be visibly obvious to the user. Except for the browser location bar (which can be different), the grader should see a page that looks as it normally does when the grader visits their profile page. No changes to the site's appearance or extraneous text should be visible. Avoiding the red warning text is an important part of the attack. It is ok if the number of bitbars displayed or the contents of the profile are not correct (so long as they look normal). It's also ok if the page looks weird briefly before correcting itself.

Put your answer in a file named `a.txt`.

**Hint:** try adding things like random text to the end of the URL. How does this change the HTML source code loaded by the browser?

## Attack B: Cross-Site Request Forgery
Your solution is a short HTML document named `b.html` that the grader will open using the web browser. The grader will already be logged in to Bitbar before loading your page. Transfer 10 bitbars from the grader's account to the attacker's account. As soon as the transfer is complete, the browser should redirect to `http://pages.cs.wisc.edu/~ace/cs642-spring-2016.html` (fast enough that a casual or distracted user might not notice). The location bar of the browser should not contain `localhost:3000` at any point.

The framebusting code may make this attack harder than necessary. For this attack - and **ONLY** this attack - you may disable framebusting on a page by adding `disable_fb=yes` to a URL. For example: `http://localhost:3000?disable_fb=yes`.


## Attack C: Clickjacking

Your solution is one or two HTML documents named `c.html` and (optionally) `c2.html`. The grader will open `c.html` using the web browser. The grader will already be logged in to Bitbar before loading your page. The grader will interact with the web page in a way that is reasonable. This means that if there is a button on the web page for the user to click, the grader will click it. The same goes for any other interaction on the page. After interacting with the page, 10 bitbars should be transferred from the grader's account to the attacker's account. As soon as the transfer is complete the browser should be redirected to `http://pages.cs.wisc.edu/~ace/cs642-spring-2016.html`.

Your attack must work by involving user interaction (i.e. do NOT just do another CSRF attack for this). In particular, your attack must target the pages `http://localhost:3000/protected_transfer` and/or `http://localhost:3000/protected_post_transfer`, which have XSRF protection enabled. You may NOT directly call `http://localhost:3000/transfer` or `http://localhost:3000/post_transfer` in any way for this attack.

It should not be obvious that your page is loading content from `http://localhost:3000`. There is some basic framebusting code that protects the application from this sort of attack. You must come up with a way of bypassing this defense. You may NOT use the disable `disable_fb=yes` parameter you may have used in the previous attack. Make sure to test your page layout after resizing your browser to make sure the attack will work for reasonable page resolutions (we will test using a minimum resolution of 800x600).

## Attack D: Profile Worm
Your solution is a profile that, when viewed, transfers 1 bitbar from the current user to a user called attacker and replaces the profile of the current user with itself. Submit a file named `d.txt` containing your malicious profile. Your malicious profile should include a witty message to the grader (to make grading more amusing).

To grade your attack, we will cut and paste the submitted profile into the profile of the attacker user and view that profile using the grader's account. We will then view the copied profile with more accounts, checking for the transfer and replication.

The transfer and application should be reasonably quick (under 15 seconds). During that time, the grader will not click anywhere. 

During the transfer and replication process, the browser's location bar should remain at `http://localhost:3000/profile?username=x`, where `x` is the user whose profile is being viewed. The visitor should not see any extra graphical user interface elements (e.g. frames), and the user whose profile is being viewed should appear to have 10 bitbars.

You will not be graded on the special case where the user viewing the profile has no bitbars to send.

**Hint:** The site allows a sanitized subset of HTML in profiles, but you can get around it. The MySpace vulnerability may provide some inspiration.

## Deliverables
1. Create files named `a.txt`, `b.html`, `c.html`, and `d.txt` containing your attacks (you can optionally include a file `c2.html`). 

1. Create a file named `README` that includes the UWID and names for each group member (there is no special format required). You may also include descriptions of how each attack works. This may be used for partial credit in some circumstances, though partial credit will not normally be given.

1. Put these files into a single directory named `attacks` and package them into a tarball with the following command (where UWID1/2 are your group members' UWID #s):
```
tar -cf UWID1_UWID2_hw2.tar attacks/*
```

Upload `UWID1_UWID2_hw2.tar` to the hw-2 folder on the Desire2Learn website for this course:
https://uwmad.courses.wisconsin.edu/d2l/home/3199130

Use Assignments > Dropbox to find the hw-2 submission folder.

## Grading
Each attack is worth up to 2 points. If the attack works, then one will receive full credit. In rare cases, partial credit may be given.

**Beware of race conditions:** Depending on how you write your code, any of these attacks may have race conditions that affect the success of your attacks. Attacks that fail on the grader's browser during grading will receive less than full credit. To ensure that you receive full credit, you should wait after making an outbound network request rather than assuming that the request will be sent immediately.

## Collaboration Policy
You are encouraged to use the internet, the Piazza discussion board for this class, and classmates for information about tools and setup. Please don't discuss solution specifics with anyone beyond your project partner.

## Acknowledgements
Much of this assignment is borrowed from a Stanford CS 155 homework assignment. Thanks for their hard work!
