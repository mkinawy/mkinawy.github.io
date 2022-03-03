---
layout: post
title: Single File Process Architecture
excerpt: Over the past 3 years or so, I developed the "Single File Process" design pattern, to help me and my teams getting faster in developing new projects and features, yet keeping the code clean, easy to understand and maintainable.
feature-img: "assets/img/feature-img/sfp.jpg"
thumbnail: "assets/img/feature-img/sfp.jpg"
tags: [.NET Core, Architecture, MediatR, Coding]
---

## Introduction

After working for lots of companies, I keep witnessing the same development problems again and again. After tons of videos and readings on CQRS, the mediator pattern, specifically the amazing [MediatR package](https://github.com/jbogard/MediatR), and with thousands of working experience, I came up with a nice solution.

Over the past 3 years or so, I developed the "Single File Process" design pattern, to help me and my teams getting faster in developing new projects and features, yet keeping the code clean, easy to understand and maintainable.

I tested this design in several projects in different sizes, with developers from several backgrounds and experties, and it was a success. A junior developer once told me that the "Single File Process" helped him a lot getting up to speed with his team and it was quite easy to be productive and still producing clean code in a quite complex system.

This article is my attempt to explain what "Single File Process" is, what it tries to solve, how it puts structure in your code, and how things could get much easier and smoother.

## Problems

Let’s start with a small list of problems I faced over and over again, which were my motivation to find a solution. 

Those are some of the problems I’m trying to solve, and you can still derive many more out of them. At the end, they all end up creeping in your code, adding more dept that costs you more, and hence cutting from the profitability of your app.

### Magic

Using some packages that inject themselves in the request/response pipeline could make you more productive, yet they are very hard to debug, not easy to understand, and have lots of side-effects.

Usually as a developer, I want to debug my code sequentially and see everything going on, starting from the request and ending with the very last response, but with things like "[AutoMapper](https://automapper.org)" and "[FluentValidation](https://fluentvalidation.net)" you would see things "magically" happen, and finding the code responsible for such is quite a challenge in big projects where lots of mappers and validators are usually put together.

### no-KISS

Our code is our poem, we are more than developers, we are artists. That could be true to an extend, but basically we are hired to deliver value, to build things in a justified way in time.

Sometimes we lose this campus and go wild, sometimes too academic or by the book, which could be a good thing, but could also mean bad maintainability due to complexity, and that’s more damaging than writing “beautiful” code.

I worked in a company that took me, and that was the average there, almost 2 months to be fully onboarded. Besides setting up my development environment, getting access to company’s stuff, etc, understanding the code was very challenging, and I’m a senior developer. Make the math and imagine a new junior joiner, multiply the numbers at best without even counting the frustration and pressure.

The code was too academic, too beautiful, for a very simple application.

### Pollution

With the MVC pattern everywhere, we tend to put Model classes in the ASP.NET project "Models" root folder, and every time we need a small change in one or introducing a new endpoint, we find tons of other models that look alike.

It’s common to see models like UserDto, UserModel, User, CreateUser, UpdateUser, UserDetail, UserDetails, etc, And all hell breaks loose when we try changing one of them, let’s address this in the next bullet.

### Reusability -f

I don’t know why we tend to -f (force) reusability, we think it’s a good achievement and it saves time hence costs. But in reality, any reusable block of code tends with time to favor one use over another, and we pay even more tech-dept costs to break this reusability in the long run.

Besides, I might accept the fact that we write “some” logic to be reused, but I don’t accept request Models to be reusable, every endpoint/API should have its own Request types. If we have endpoints that receive the same exact request, then we have a bad business design.

### Believes

Systems are becoming more and more complex, and users are expecting more and more from us. To face this challenge with limited time and labor, developers need to be more “mechanical”, give more focus to the business and the value we deliver to our clients, more thinking about the features users need, and how to coin this into code.

But, there are others who have different believes, they have their own technical preferences of how we should organize and write our code. Those developers tend to lose the vision pretty fast, and their value drops quite dramatically, their code turns obsolete in no time yet it looks pretty. Tunnel vision is an example of this case where developers focus on the wrong thing.

## Single File Process

Enough said, lets drill into the details of the “Single File Process”. Basically, it’s a single class that represents a single business process. Making an order is a process, resetting your password is a process, fetching page #3 of the products is a process, you got the idea.

Every process class contains 5 nested public classes, in this order:

1. `Request` class _(required)_: this class represents the coming request, with data coming from the body, the URL, headers, etc. passed in.
It inherits from `IRequest<Response>` (from **MediatR**)
1. `Response` class _(optional)_: this one represents the output of the process, for example a list of products, or just a boolean that indicates the success of the operation
1. `Handler` class _(required)_: this is the engine that runs the process, it converts the inputs `Request` into outputs `Response` by doing the necessary work.  
This is the one that occupies most of the space of the parent process class, which gives clear indication on how complex your operation is.  
It inherits from `IRequestHandler<Request, Response>` (from **MediatR**)
1. `Validator` class _(optional)_: if you want to validate anything regarding the coming `Request`, this is the place for it. You can of course do this in the `Handler`, but I usually use **FluentValidation** for such, it’s still magic but at least it’s organized and I can see it in the same process scope.
1. `Mapper` class _(optional)_: if you want to transform `Request` to `Response`, or converting others in the `Handler`, this is the place to do it. You can also do this inline in the `Handler`, but I use **AutoMapper** for such, and at least I have the related mapping code in the same process scope.

With these 5 classes in place and by only looking at the process class, you can easily understand everything related to the process in a single file, you don’t need to look elsewhere. Everything related to this business process is encapsulated in this singe file.

## Hello World

To make it clear, let’s work with an example. Reset password could be a nice one here. Typically, user resets his password by passing in his email address as an input and expects an email being sent to his email inbox with a link for assigning a new password.

### ResetPasswordProcess

In this example, the "Single File Process" `ResetPasswordProcess` for this case will be similar to the following:

```c#
using System.Threading;
using System.Threading.Tasks;
using FluentValidation;
using MediatR;

namespace SingleFileProcessDemo.Processes.Account
{
    public class ResetPasswordProcess
    {
        public class Request : IRequest<Response>
        {
            public string Email { get; set; }
        }

        public class Response
        {
            public bool Success { get; set; }
            public bool EmailSent { get; set; }
        }

        public class Handler : IRequestHandler<Request, Response>
        {
            public Task<Response> Handle(Request request, CancellationToken cancellationToken)
            {
                // do some authentication ...
                // retrieve from db ...
                // generate reset link ...
                // send email to user ...

                return Task.FromResult(new Response
                {
                    Success = true,
                    EmailSent = true,
                });
            }
        }

        public class Validator : AbstractValidator<Request>
        {
            public Validator()
            {
                RuleFor(x => x.Email).NotEmpty();
            }
        }
    }
}
```

### AccountController

In case of handling a user request, you will typically trigger the process from the corresponding controller. For our example of resetting password, it would be something like this:  

**Note**: notice the `ResetPasswordProcess.Request` parameter, and the `response` will be of type `ResetPasswordProcess.Response`
{: .note}

```c#
using System.Threading;
using System.Threading.Tasks;
using MediatR;
using Microsoft.AspNetCore.Mvc;
using SingleFileProcessDemo.Processes.Account;

namespace SingleFileProcessDemo.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class AccountController : ControllerBase
    {
        private readonly IMediator _mediator;

        public AccountController(IMediator mediator)
        {
            _mediator = mediator;
        }

        [HttpPost("reset-password")]
        public async Task<IActionResult> ResetPassword([FromBody] ResetPasswordProcess.Request request, CancellationToken cancellationToken)
        {
            var response = await _mediator.Send(request, cancellationToken);
            return Ok(response);
        }
    }
}
```

Notice here that `ResetPasswordProcess.Request` won’t be used anywhere else, it’s quite bound to this, and only this, endpoint. This helps a lot in debugging as you can clearly see who is responsible for what.

You don’t need anything extra injected in your controller, except the `IMediator`. This makes your controller very thin.

This controller example is basically what you would do in all cases. Typically, a `Request` being added as an action parameter and flagged as `FromBody`, `FromRoute`, etc, and then pass such request to the `IMediator.Send` method, and handle the results, which will be of the defined `Response` type.

### IHostedService

Same like `Controller`, if you are handling a background job, you would do exactly the same thing, except of course that you need to initialize your own `Request`, send it via `IMediator.Send`, and finally handle the results. Basically the same, but instead of having the `Controller` to be the process host/trigger, it would be your `IHostedService`.

## Processes overview

I always group my process classes in a root folder called “Processes”, which inside I include one level of feature-based folder, something like “Processes/Account” or “Processes/Products”.

By implementing the "Single File Process" pattern and organizing your code like this, you can easily at glance see what kind of business workflows your app handles. for example:

![Processes overview](/assets/img/posts/sfp-files.png "Processes overview")
{: .text-center}

This will help you also finding the responsible process easily and understanding the code quickly, yet better “onboarding”.

## Closing thoughts

- I like adding the suffix “Process” to each process class, as I don’t like ending up with a verb-based class like “GetProfile“. Actually, I like adding suffixes to every class/type I have according to its purpose, that way it’s much easier for me to know exactly what it does. Things like: `AccountController`, `UserEntity`, `PageModel`, `AuthConstants`, `CategoryService`, etc. And I like grouping them in their own folders, say “Controllers”, “Services”, “Entities”, “Constants”, “Models”, etc, besides of course “Processes”.  
This helps me a lot being productive, fast in finding the file I’m looking for, and easy to understand for everyone.
- By checking the size of the `Handler` class in every "Single File Process", I know immediately if my process is in the right manageable code size, or I might need to rethink and redesign.  
Sometimes I break the code in the `Handler` into multiple inner private methods, sometimes I break a huge process into multiple, it all depends on the business, but the thing that I try my best, is avoiding over-fitting.
- Unit testing becomes much easier and direct. By focusing on testing the `Handle` method of the `Handler` class, you will cover most of your code and almost all of your business.
- "Single File Process" is not a silver bullet, yet I didn’t face a case that it cannot manage, but if that happens to you, feel free to skip. I can think of things like Saga’s and Workflows where huge lines of code and big status check trees need to be written together. Those cases will be little though, and I still think they could be transformed to a better sized processes. The "Single File Process" gives you great structure and important indicators about your business and process sizing  
- I used the Process keyword here as it’s not common in the web development world, besides, every "Single File Process" handles a single feature unit in your app, that’s why I found Process a nice term.
- Besides `ASP.NET Core` and `MediatR`, in totally different technologies or languages, I still think you can benefit from the "Single File Process" pattern and it could be implemented easily. At the end, it’s a CQRS implementation with Mediator pattern in a single file represents a small business unit.
- "Single File Process" is a dramatic change in our way of coding. Give it sometime and take it easy, try it out in a small project and see how it goes. Changes could be scary, and I heard lots of rejections the first time I introduced this to colleagues, but later they saw the benefits. Give it time!

## Conclusion

"Single File Process" pattern is a way to encapsulate business feature units into corresponding process files, making it easy for everyone to understand and be productive. It puts things in order, forces a structure, makes your code clear, easy to understand, easy to maintain and easy to grow fast.

I got inspired a lot by this talk of **Jimmy Bogard** (MediatR author) on "Vertical Slice Architecture", I highly recommend watching it:

<figure class="video_container">
  <iframe src="https://www.youtube.com/embed/T6nglsEDaqA" frameborder="0" allowfullscreen="true"> </iframe>
</figure>

<br />

You can find a code example on "Single File Process" in this repo, I included more complex cases:  
<https://github.com/mkinawy/single-file-process-demo>

I hope you enjoyed reading this, feel free to give your feedback, I would love hearing your thoughts on this.
