---
title: "Finding a vulnerability when you just want to buy a car"
date: 2023-06-07T02:01:58+05:30
description: "Read my story on trying not to get arrested, spoiler: haaa ya thought!"
draft: true
---

About 2 months ago, I was browsing the car auctions for broken BMWs on a saturday night, as you do. It's well known that all Australian auction platforms compete with each other on who has the worst website, according to literally everyone I know that has used one (or more) of these sites. 

Anyways, that's not really the point of this post, we are supposed to be talking about [hacker stuff](images/hac.jpg).

## Saturday night enjoyers 👀

I am on [grays.com.au](https://www.grays.com/) looking at a listing for one of the said BMWs, and realise there's been like 130 something bids on it so far, who are all these people spending their saturday night looking at cars that don't run? Why aren't they outside, socialising, having a good time? weirdos.

{{< figure src="images/weirdos.png" class="center-figure" title="list of said weirdos" >}}

The "bidding details" are the bidder's initials and their suburb, why do we need to see the initials of every single bidder? I mean it has no use other than to see who you are in a bidding war with. Maybe it's a psychological tactic to drive up the egos of bidders so they can go like "yeah I will beat J.K this time around"?

Being able to see the initials got me thinking 🧠, surely I can see the full names, right? I wanted to see if there were juicy things hidden *inside* the page. To do it, I had to use the *only* hacker tool I know.

{{< figure src="images/hacking-tool.png" class="center-figure" title="Right click > Inspect, all you need to subvert any website known to man" >}}

Clicking this will launch the developer tools of your browser, you can see what kinds of network calls the website is making. 

## Meh....
So we know a few things, they have some sort of a database to store all bidder details, they MUST know the full name of the bidders (because they have the initials, *duh*) and they are somehow getting this data from the database and displaying it on the website for your eyeballs to see. 

While we have the inspect element open (and on the 'network' tab), we see the network calls being made as soon as we click the 'view bidders' button. One request returns a list of bidder information for that listing

```javascript
"Bids": [
    {
    "LotId": 20982268,
			"UserId": "379e6d8f-7608-4d38-b10f-7330e7c03c7c",
			"UserInitials": "M.W",
			"Date": "/Date(1686147660227)/",
			"Price": 27250,
			"Quantity": 1,
			"WinningQuantity": 1,
			"OriginalDate": "/Date(1686135239583)/",
			"MaximumBidPrice": 0,
			"UserShortAddress": "Wembley WA",
			"FormattedDate": "08/06/2023 <a href=\"https://www.grays.com/changetimezone.aspx?ReturnUrl=%2flot%2f0001-9041855%2fmotor-vehiclesmotor-cycles%2f2005-toyota-landcruiser-gxl-4x4-hdj100r-t-diesel-automatic-8-seats-wagon%3fspr%3dtrue\" title=\"&#40;UTC&#43;10&#58;00&#41;&#32;Canberra,&#32;Melbourne,&#32;Sydney\">12.21.00 AM AEST</a>",
			"FormattedPrice": "<span class=\"currency\"><span class=\"abbr\"><span title=\"AUD\">AU $</span></span>27,250</span>"
	},
    ....
]
```

At a first glance, I don't reallyyy get anything out of this, `UserInitials` is there, but nothing else that seemed "oh no" worthy[^1]

## More clicking around

While we have mr.inspector open, why don't we just browse the website around? I'm just going to click 1 link, just 1 more. 

Clicking 'My Grays' will show you a page with a form, a form prefilled with all your details when you signed up; full name, email and all the other *private* details. All of this is being fetched how?

We make another network call to `account/yourdetails.aspx`. With this request, you have to specify exactly *who's* details you want to get. We can see that through the request headers.
``` javascript
GET /mygrays/account/yourdetails.aspx HTTP/2
2 Host: www.grays.com
3 Cookie: forterToken=80fc631a26ff4aa481958f63433821d_1682220260323_715_UAL9_11ck; deviceScreenSize=xl;
deviceSmallScreenSizeSet=0;AMCV_grays$40Adobe0rg=T;s_fid=71D698F8C5D8A0EE-2C2EA6F58CA0BB37;5_vnuM= 1704890873504826vnS3D17;
_zUcmid=1Dr lW2Au3Sd155K; ItemsPerPage=100; Murray_Identity=
Ka19664a9-694c-412d-9507-ef6ce2f0flae}20230423T032451:20; _nr=1682220300169-Repeat;visited=1;
GlobalGraysTrackingData=
§7B$22RecentlyVisitedPage$2283A$7BS22EntityId$2283A$22%22%2C822EntityCategoryId$22%3A$22automotive-trucks-
land-marines2282C822EntityCategoryType 2283A822Industrial®22%7D$7D; RequestCorrelationId=
(28b673a8-1bbf-4df8-90bc-7c7228dae006; Murray_TimeZone=AUSEasternStandardTime;s_ev50-Browse;5_5q=
grays-prds3D$2526c.$2526a.$2526activitymap.$2526pages253Dhttps$25253A$25252F$25252Fwww.grays.com$25252F$25
|261ink$253DMYS252520GRAYS$2526region$253Dlinks$2526.activitymap$2526.a%2526.c$2526pid$253Dhttps$25253A8252
52F$25252Fwww.grays.com$25252F$2526oid%253Dhttps$25253A825252F$25252Fwww.grays.com$25252Fmygrays$25252Facc
lount&25252Fyourdetails.aspx82526ot8253DA;
Murray_UserLastModDateTime=2023-04-2211:12:36Z;s_cc=true;s_invisit=true;s_ppn=
https83A82F$2Fwww.grays.com$2F;
```

I looked at this for about 30 minutes before I saw it. Can you see it? CAN YOU?

```javascript
Murray_Identity={a19664a9-694c-412d-9507-ef6ce2f0f1ae}
```

This looks just like the UserID we saw from the other request, the one to get the bidder details 😳

So what if I just replace my own `Murray_Identity` [^4] to ANY of the ones I got from the list of bidders and forward the request? Do I try? I have to, right? I mean I'm sure it won't do anything and will just redirect me somewhere else... right?? Using Burp Suite (another elite hacking tool), we can intercept the request, change the `Murray_Identity` and forward it like nothing happened.

## Crimes(?)
{{< figure src="images/muh-details.png" class="center-figure" title="Me looking at things I shouldn't be looking at? maybe?" >}}

**oh no**

It's just *there*.

At this point I was fairly sure I was looking at the some information that was not *meant* for me to see and I was *kinda* worried that I was somehow doing something wrong, but like, not enough to stop.

**...anything *else* in this page?**\
Well, damn if all this info is in this treasure trove of computer spaghetti, maybe there's wayyyy more. Perhaps this HTML contains the lost launch codes to the Sydney Opera House, or Harold Holt [^2]

What if I click on the "Payment" tab and intercept the request like did I above, surely they must have some extra security 🔐 on veeerrry sensitive data?

{{< figure src="images/money-tab.png" class="center-figure" title="This is better, they seem to be only fetching the last 4 digits of card numbers, no pre-population of field going on here" >}}

Does this mean I can go through all the pages one by one, as if *someone else* is logged in? I mean I *wanted* to, but I didn't [^3]

## What have i done
Googling for "how many customers does grays have?" returns [3 million customers](https://www.grays.com/content.aspx?block=OpenForBusiness). Obviously only the people who actually *bid* on items were exposed to this vulnerability. But that's still *a lot* of people.

I'd now found people's:
- Full names
- Phone numbers
- Home addresses
- Last 4 digits of credit cards

## What else?

The website periodically sends out a request called GetLoginStatus, this is to check if you're still logged in, so that it can notify you if you have been outbid on the not working car you want to buy. Replacing the `Murray_Identity` [^5] header in the request returns *stuff*.

{{< figure src="images/more-info.png" class="center-figure" title="oh_no_again.jpg" >}}

Page refreshes and i'm given a whole bunch of data that I wasn't supposed to be given. Hold on....

{{< figure src="images/login-horror.png" class="center-figure" >}}

I WAS LITERALLY LOGGED IN AS SOMEONE ELSE.

Intrusive thoughts took over and I had a sudden realisation, I could buy some poor soul this monstrosity and they couldn't do anything about it.

{{< figure src="images/ugly-car.avif" class="center-figure" title="i could have bought someone a bathtub on wheels" >}}

By this point I'd had enough clicking around and was like *oh jeez oh boy oh jeez*. *I gotta get someone somehow to take a look at this*. I wasn't just going to email grays "hey i found thousands (millions?) of people's leaked info on your website", because [that's how you go to jail](https://www.bleepingcomputer.com/news/security/ethical-hacker-exposes-magyar-telekom-vulnerabilities-faces-8-years-in-jail/).

## Disclosing the problem
I've always thought about the issue and complexity of disclosing a vulnerability when you *accidentally* find it:
- how do i contact the company?
- what do i say?
- have i done a crime?

First thought I had is to ask my fellow software engineer friend, i don't really have a good explanation for this so i'm just gonna post the screenshots.
{{< figure src="images/friend-advice.png" class="center-figure" title="The planet may be dying, but we live in a truly unparalleled age of content." >}}

He suggested I take up a lawyer's advice, followed by... "just ask reddit bro". I contacted *a lot* of people about this. If my calculations are correct [^6], I called at least 10 friends and other people in the *hacker industry* who may have the slightest clue on what to do [^7]

## trying to ask a lawyer if I gone and done a crime
Before I went and told everyone about my HTML frolicking, I spent like 3 days calling legal aid numbers, lawyers, and otherwise trying to figure out if I'd done a crime [^8].

During this time, I didn't tell *anyone* what I'd done. I asked if any laws would be broken if "someone" had "logged into a website accidentally using someone else's publicly available info and found personal information". Do you see how that's not even a lie? I'm starting to see how lawyers do it.

## asking an actual professional for help

Then I remembered about [Troy Hunt](https://www.troyhunt.com/), he knows the [pain of disclosure](https://www.troyhunt.com/breach-disclosure-blow-by-blow-heres-why-its-so-hard/), surely he can help me out.

I prepare a *professionally* written email to Troy explaining what my eyes had seen, and await his response.

{{< figure src="images/troy-email-1.png" class="center-figure" title="Email response from Troy, who thinks I am not scared of jail" >}}

Yeah that's not gonna work Troy. Too many horror stories of innocent(?) security researchers ending up in jail because they want to *responsibly* disclose vulnerabilities.

{{< figure src="images/troy-agrees.png" class="center-figure" title="he gets it" >}}

So I tell him exactly what had happened, and how to replicate what I had done, basically what you've read so far in this here blog post. And he got back to me almost immediately.

{{< figure src="images/proud-troy.png" class="center-figure" title="wait, LinkedIn is actually useful for once?" >}}

{{< figure src="images/grays-email.png" class="center-figure" title="email from an important person telling my the problem is no longer a problem anymore (not fully)" >}}

## Closing credits

### How it works
The entirety of this vulnerability stems from exposing the unique user identifiers while viewing the bidding history for an auction lot. Without the user identifier, it would not be possible to swap the cookie session [^9]. Updating the session cookie based on one request header is a silly idea.

### Timeline of events
- Apr 22 - Issue found
- Apr 22 - I realise the issue is much bigger than it is
- Apr 23 - I learn from lawyers that I have not done a crime 💯
- Apr 23 - I contact Troy Hunt in hopes of him assisting me with resposible disclosure
- Apr 24 - Contacted important guy from Grays
- Apr 25 - First part of issue fixed
- Apr 26 - All parts of issue fixed
- Apr 30 - I turn 25 🥳





[^1]: not that I was hoping to find anything i swear
[^2]: Harold Holt was a former Prime Minister and we... lost him? He disappeared while going for a swim one morning. This is not a joke. We named [Harold Holt Memorial Swim Centre](https://en.wikipedia.org/wiki/Harold_Holt_Memorial_Swimming_Centre) after him. I repeat, this is *not* a joke.
[^3]: you'll have to trust me on this
[^4]: Whatever that means
[^5]: Still not sure what it means
[^6]: I've always wanted to say that
[^7]: They didn't
[^8]: I'm not really sure what my plan was. If I had done a crime, what was I gonna do, not report the publicly available sensitive data of thousands of people?
[^9]: Because you wouldn't have the user identifier
