---
layout: post
title: Scraping JavaScript websites with .NET Core
excerpt: At work, I needed to find a way to scrape a website and gather data required by some team, and that is quite trivial task IF the website is just a normal one with pages that could be fetched/downloaded using regular HttpClient/WebRequest calls and parse the content via tools like HtmlAgilityPack, but that wasn’t the case, the website was built with Angular, and it makes lots and lots of Ajax calls.
feature-img: "assets/img/feature-img/html.jpg"
thumbnail: "assets/img/feature-img/html.jpg"
tags: [.NET Core, JavaScript, Scraping, Coding]
---

## Intro

At work, I needed to find a way to scrape a website and gather data required by some team, and that is quite trivial task IF the website is just a normal one with pages that could be fetched/downloaded using regular `HttpClient/WebRequest` calls and parse the content via tools like `HtmlAgilityPack`, but that wasn’t the case, the website was built with `Angular`, and it makes lots and lots of Ajax calls, and then aggregate the data and render them in a meaningful way.

JavaScript-based websites require a different approach for scraping, you need to treat them exactly like an end user does, via browsers, and that exactly what I did.

The other option beside automating this process, is by hiring someone doing all this manually, tinkering with the website DOM and the browser console to retrieve the displayed data, and the hidden info like the items’ IDs. This is quite hectic, so boring, and quite error prone.

So the decision was to automate this, and I already have some good experience with this topic, but that was on .NET Framework WinForms and Node.js with Electron, and I would prefer to build this using the latest .NET Core 2.x, which I’m going to show you in a minute.

Besides that, I want to develop on my MacBook machine, using my favorite IDE, Rider, so the solution needs to work cross-platform.

Also, it’s important for me to be able to test my algorithm, is to see the actual browser and the actual behaviour of my code tinkering with the website live.

I started by running some research, and I was disappointed a lot, most of the resources I found were about the old .NET Framework, very little are about .NET Core, and even those are focusing on integration testing rather than scraping, so "headless" browsers are more appreciated there.

In the coming few lines, I will go through each option I found, and the pros and cons of each, and will go deeply over the option I picked.

## Option 1: WebBroswer

`WebBroswer` is a `WinForms` control that you can use for rendering HTML or just navigating to URLs. It’s basically a wrapper for your local IE, but it has quite aggressive compatibility which makes the rendered results inside it look weird, different, and probably suffer from lots of JS errors, specially if the website is built using quite recent and modern JS framework/library.

For the above reasons, I decided to skip this option and just keep it as my last resort, specially I don’t think that .NET Core WinForms is mature enough, besides of course the other obvious reasons of how bad it is for using IE for browsing and scraping.

## Option 2: PhantomJS

`PhantomJS` is a headless web browser scriptable with JavaScript. It runs on Windows, macOS, Linux, and FreeBSD. Paired with `Selenium`, you get the job done!

This was my initial choice due to my experience and maturity of the platform, [PhantomJS](https://phantomjs.org) is a quite famous in the field, `Selenium` and `PhantomJS` are like the perfect and well known pair for the job.

But, on March 2018, `PhantomJS` guys decided to suspend further development due to the decrease in contributors and in the community interest, you can read more about this [here](https://github.com/ariya/phantomjs/issues/15344). This means no more bug fixes, and of course lack of recent browser capabilities support.

Though, I still decided to give it a try. After installing the latest version of `Selenium.WebDriver` nuget package in a new .NET Core Console application, I found that they stripped out all `PhantomJS` references, for example the `PhantomJSDriver` class is not there any more, so I decided to downgrade my `Selenium` version further down, till I found the classes I was looking for, but with big warnings that those classes are obsolete and are going to be removed in next versions, then I downgraded further, v1.10 to be exact, till I felt that `PhantomJS` is fully supported by `Selenium`.

The developing process was straight forward, by initiating `PhantomJSDriver` by passing the bin location of my OS `PhantomJS` which I downloaded from <https://phantomjs.org/download.html>

Since it’s a headless browser, I had to do lots of trial and error, and the first issue I faced was getting warnings from the target website server that I’m using outdated browser and I need to upgrade to get the best user experience, `PhantomJS` sends old user-agent information, and that could be overwritten in the code, but that wasn’t my biggest problem.

The biggest issue was reading the nested nodes/elements of a custom HTML component tag, it’s quite normal in `Angular` application to see tags like `<app-root>` and `<router-outlet>`, and those are not really standard HTML tags, and `PhantomJS` was facing bad times trying to read them and their children.

So the conclusion was a big "No", sadly enough due to the good times I had with this great tool. It’s time to move on.

## Option 3: ChromeDriver

It took me sometime to update my knowledge and learn this new kid. Also I heard about a similar one for FireFox, but I decided Chrome would be a good fit.

The process is very straight forward, you just need to install these two nuget packages, lets say in a new .NET Core Console app:

- `Selenium.WebDriver`: latest version
- `Selenium.WebDriver.ChromeDriver`: latest version is OK, but you need to check the Chrome version installed on your machine, and you might need to downgrade this one or update your Chrome. This package puts the necessary executable files in your project output folder, so you don’t need to download any custom ones, if you do and want to skip this package, you can still download the target version from <http://chromedriver.storage.googleapis.com/index.html>

That’s really all you need. After that it’s time for writing some code:

```c#
// finding the chrome executable path (bin)
var path = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
var driver = new ChromeDriver(path);
 
driver.Navigate().GoToUrl("https://js-website.com);
 
// doing some web storage manipulation
driver.ExecuteScript("localStorage.setItem('language', 'en');");
driver.ExecuteScript("localStorage.setItem('lat', '30');");
driver.ExecuteScript("localStorage.setItem('lng', '30');");
 
// refreshing is important for the website js to handle the updated localStorage
driver.Navigate().Refresh();
 
// sleeping/throttling waiting for everything to settle
Thread.Sleep(10 * 1000);
 
var header = driver.FindElementByCssSelector(".navbar-brand");
Console.WriteLine("Header: " + header.GetAttribute("innerHTML"));
 
var items = driver.FindElementsByCssSelector("div.container span.item");
Console.WriteLine("Items Count: " + items.Count);
foreach (var item in items)
{
    var title = item.Text;
    Console.WriteLine("Item: " + title);
    if (string.IsNullOrWhiteSpace(title) || title == "All Items") continue;
 
    item.Click();
    Thread.Sleep(5 * 1000);
 
    // continue...
}
```

That’s it basically, you can put this code inside your Console `Main` method, or organize the code the way you like.

This option has been my choice obviously. When I run this code, it initiates a Chrome instance in front of me and I see how my code interacts with the website. Also still you can set some capabilities to make it for example a headless browser, besides some other options specific to Chrome.

## Final Notes

- It’s very important to do throttling between certain actions in your code, giving the website’s JavaScript the time to do its work, it’s quite difficult to detect programmatically when the website’s JavaScript is doing or done with something, that’s why sleeping could be your best option, specially due to inconsistency, lots of JavaScript libraries and non-standard code that you don’t have control over
- Hitting the same server from the same IP too much, could be flagged automatically by recent servers as a DoS attack, and you might even get your IP blocked, so throttling and slowing down the process and running only through a single thread to resemble a normal human user interaction, would be beneficial
- Slowing down the process helps me a lot to monitor my code, at least during development/debug times, so the sleeping time could be a setting maybe if you want to publish your scraper
- You need to have your compliances in check, legal and alike agreements could be needed when you need to scrape others’s websites, respecting their ownership over their data and their conditions. In this specific case I demonstrated, the target website is actually owned by the same company but the designated team is not working with the company anymore, so it’s much easier just to scrape than running into cycles of discussions and developing and fixing
- You need to be smart. Maybe instead of writing code for clicking on several elements in a certain order to reach a certain page, you can do the same by just storing something in the `localStorage` or in a cookie. You need to study your target website and notice it’s behaviors
- Web scraping requires good experience on both the backend and the frontend, and how the web works and how browsers handle them

That’s it, I hope you enjoy it and hopefully you learn something new, looking forward to your feedback and any questions. Happy coding!
