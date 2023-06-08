---
title: "Finding a vulnerability when you just want to buy a car"
date: 2023-06-07T02:01:58+05:30
description: "You have a to-do list that scrolls on for days. You are managing multiple projects, getting lots of email and messages on different messaging systems, managing finances and personal health habits and so much more."
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

At a first glance, I don't reallyyy get anything out of this, `UserInitials` is there, but nothing else that seemed "oh no" worthy <sup>(not that I was hoping to find anything i swear)<sup>.

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

So what if I just replace my own `Murray_Identity` to ANY of the ones I got from the list of bidders and forward the request? Do I try? I have to, right? I mean I'm sure it won't do anything and will just redirect me somewhere else... right?? Using Burp Suite (another elite hacking tool), we can intercept request, change the `Murray_Identity` and forward it like nothing happened.

## Crimes(?)
{{< figure src="images/muh-details.png" class="center-figure" title="Me looking at things I shouldn't be looking at? maybe?" >}}

I guess I was now looking at private info of some guy named Mark<sup>that wasn't the actual name i was looking at</sup>